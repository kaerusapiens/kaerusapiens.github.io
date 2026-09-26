---
# 포스트 기본 메타데이터 설정
title: "GCP 보안 운영: Security Command Center(SCC), SHA vs ETD, 그리고 맞춤형 위협 탐지"
# 발행 일시 (타임존 +0900 명시)
date: 2026-09-23 16:15:00 +0900
# 카테고리: 대분류(GCP)와 소분류(Security)로 구조화
categories: [GCP, Security]
# 수식 렌더링 여부
math: false
# 메인 상단 고정 여부
pin: false
# 목차(Table of Contents) 활성화
toc: true
---

## 1. 개요 (Overview)

Google Cloud의 **Security Command Center (SCC)**는 클라우드 자산의 구성 오류, 취약점, 비정상 행위를
중앙에서 실시간으로 감시하고 분석하는 핵심 보안 관제탑입니다.

기본 탑재된 탐지기(Built-in Detectors)만으로도 수많은 보안 위협을 잡아낼 수 있지만, 기업마다 고유한
규제(PCI DSS, HIPAA 등)나 내부 보안 정책을 적용하려면 **Security Health Analytics(SHA) Custom
Module**을 직접 작성해야 합니다.

이 글에서는 커스텀 모듈의 3대 핵심 구성 요소인 **`resource_selector`**, **`CEL Predicate`**,
**`Dynamic Mute Rule`**의 동작 원리와, 자격증 시험 및 실무에서 가장 빈번하게 실수하는 **보안 탐지
공백(Coverage Gap) 방지 아키텍처**, 그리고 **SHA(상태 기반) vs ETD(이벤트 기반)의 핵심 차이**를
정리합니다.

---

## 2. 맞춤형 보안 검사 파이프라인 흐름도

SCC에서 특정 조건의 암호키(CryptoKey)를 모니터링할 때의 표준 검사 아키텍처입니다:

```text
[ Security Command Center (SCC) ]
   │
   ├── 1. resource_selector ──▶ "Cloud KMS의 CryptoKey만 콕 집어서 골라내라!"
   │
   ├── 2. CEL Predicate     ──▶ "그중 라벨이 'pci'이고 회전 주기가 30일 넘는 것만 위험으로 판정해라!"
   │                             (위반 시 Custom Finding 경고 발생)
   │
   └── 3. Dynamic Mute Rule ──▶ "기본 탐지기(KMS_KEY_NOT_ROTATED)가 해당 키에 대해 중복 경고를 울리면
                                 자동으로 음소거(Mute) 처리해서 보안팀을 귀찮게 하지 마라!"
```

---

## 3. 핵심 용어 및 컴포넌트 상세 분석

### 3.1 Custom Module (SHA 커스텀 모듈)

- **개념**: Security Health Analytics에서 기본 제공하는 탐지 규칙 외에, 기업의 자체 규정에 맞춰 직접
  제작하는 **맞춤형 취약점 스캐너**입니다.
- **필요성**: 구글 기본 탐지기(`KMS_KEY_NOT_ROTATED`)는 90일 회전 여부만 검사하지만, 특정 금융
  결제용 키는 30일마다 회전해야 하는 등 세분화된 규칙이 필요할 때 사용합니다.

### 3.2 `resource_selector` (리소스 선택자)

- **개념**: 커스텀 모듈이 검사할 **대상 클라우드 리소스의 유형(Resource Type)**을 선언하는 타겟
  필터입니다.
- **예시**: `cloudkms.googleapis.com/CryptoKey`를 지정하면, 수천 개의 VM, 버킷, DB를 전부 건너뛰고
  **오직 Cloud KMS의 암호화 키 객체들만 정밀 스캔**합니다.

### 3.3 CEL Predicate (CEL 술어 / 조건식)

- **개념**: **Common Expression Language (CEL)**을 사용해 "어떤 상태를 보안 위반(True)으로 판단할
  것인가?"를 정의하는 논리 수식입니다.
- **평가 원리**: 수식의 결과가 `True`로 평가되면 SCC 대시보드에 즉시 보안 경고(Finding)가
  생성됩니다.
- **수식 예시**:
  ```cel
  // 라벨에 'compliance: pci'가 있고, 회전 주기가 30일(2,592,000초)을 초과하는 키를 위반으로 판정
  resource.labels["compliance"] == "pci" &&
  resource.rotationPeriod > duration("2592000s")
  ```

### 3.4 Dynamic Mute Rule (동적 음소거 규칙) ⭐

- **개념**: 보안 알림(Finding)이 발생했을 때, 사전 정의된 조건에 일치하면 알림 목록에서 **자동으로
  '음소거(Muted)' 상태로 분류**해 주는 필터 규칙입니다.
- **핵심 목적**: 보안팀이 이미 인지하고 있거나 커스텀 모듈로 별도 관리 중인 리소스에 대해, 중복
  알람이 울려 발생하는 **알람 피로도(Alert Fatigue)를 원천 해소**합니다.

---

## 4. 심층 아키텍처 분석: 공식 문서 권고와 범위(Scope)의 함정

자격증 시험과 엔터프라이즈 설계에서 수많은 엔지니어들이 함정에 빠지는 포인트입니다.

### 4.1 공식 문서의 일반적 권고

> _"커스텀 모듈이 암호키 회전을 중복으로 검사할 경우, 기본 탐지기인 `KMS_KEY_NOT_ROTATED`를
> 비활성화(Disable)하라."_

### 4.2 권고 사항의 전제 조건과 실제 위험 (Coverage Gap)

- 구글의 권고는 **"커스텀 모듈과 기본 탐지기의 검사 범위(Scope)가 완벽히 동일할 때"**를 전제로
  합니다.
- 만약 커스텀 모듈이 **'특정 라벨이 붙은 키만 필터링하여 검사'**하고 있다면:
  - 기본 탐지기를 꺼버리는 순간, **라벨이 붙지 않은 사내 모든 일반 암호키에 대한 '90일 기본 회전
    검사선'이 통째로 증발**합니다!
  - 이는 단순한 중복 제거가 아니라, 심각한 **보안 탐지 공백(Coverage Gap)**을 유발합니다.

### 4.3 올바른 엔터프라이즈 해결책

1. 기본 탐지기(`KMS_KEY_NOT_ROTATED`)는 **반드시 활성화 상태(`keep enabled`)를 유지**하여 라벨 없는
   일반 키들의 90일 기준선을 지킵니다.
2. 커스텀 모듈이 이미 30일 기준으로 엄격하게 감시하고 있는 라벨 부착 키에 대해서만, **Dynamic Mute
   Rule을 적용하여 기본 탐지기의 알람을 무음 처리**합니다.

> [!TIP] **체화해 두면 좋은 엔지니어링 습관 (The habit worth building)**  
> 플랫폼의 일반적인 '중복 제거(Redundancy) 권고'는 동일한 범위를 전제로 합니다. 리소스나 탐지기를
> 비활성화(Disable)하기 전에는, 필터 조건 때문에 실제 보안 감시 범위가 좁아져 보안 공백이 생기지
> 않는지 항상 확인해야 합니다.

---

## 5. 실무 커스텀 모듈 설정 예시 (YAML)

```yaml
# Security Health Analytics 커스텀 모듈 정의 파일
name: organizations/[ORG_ID]/securityHealthAnalyticsSettings/customModules/custom-kms-30d-rotation
displayName: "PCI-DSS 라벨 키 30일 회전 검사 모듈"
enablementState: ENABLED

customConfig:
  # 1. 검사 대상 리소스 타입 선언 (resource_selector)
  resourceSelector:
    resourceTypes:
      - "cloudkms.googleapis.com/CryptoKey"

  # 2. 위반 조건을 판정하는 CEL 조건식 (predicate)
  predicate:
    expression:
      'resource.labels["compliance"] == "pci" && resource.rotationPeriod > duration("2592000s")'

  # 3. 위반 시 보고 설정
  severity: HIGH
  description: "PCI-DSS 규제 대상 암호키의 회전 주기가 30일을 초과했습니다."
  recommendation: "gcloud kms keys update 명령어로 회전 주기를 30일 이하로 단축하세요."
```

---

## 6. Security Health Analytics (SHA) vs Event Threat Detection (ETD)

SCC의 양대 탐지 엔진인 **SHA**와 **ETD**는 탐지 대상과 Finding의 수명 주기가 완전히 다릅니다.

```text
[ 방식 A : Security Health Analytics (SHA) - 상태 기반 스캐너 ]
  외부 계정에 Owner 부여 ──▶ [ SHA 주기적 스캔: 취약점 경고 발생! ]
                                   │
  관리자가 권한 회수      ──▶ [ 상태가 정상으로 복구됨 -> Finding이 조용히 닫히고 사라짐! ❌ ]
                               * 사후 침해 사고 조사(포렌식) 불가!

--------------------------------------------------------------------------------

[ 방식 B : Event Threat Detection (ETD) - 로그/이벤트 기반 위협 탐지 ]
  외부 계정에 Owner 부여 ──▶ [ Cloud Audit Logs 발생 ]
                                   │ (실시간 스트림 감지)
                             [ ETD: "Persistence: IAM Anomalous Grant" 경고 생성! ]
                                   │
  관리자가 권한 회수      ──▶ [ 이미 발생한 보안 이벤트 기록(Finding)은 그대로 영구 보존! ⭕ ]
                               * 분석가가 수동 해결(manually resolved)할 때까지 활성 상태 유지!
```

### 6.1 핵심 비교 요약표

| 비교 항목             | Security Health Analytics (SHA)                        | Event Threat Detection (ETD)                                |
| :-------------------- | :----------------------------------------------------- | :---------------------------------------------------------- |
| **분석 대상**         | 클라우드 리소스의 **현재 구성 상태 (State)**           | **Cloud Audit Logs (실시간 감사 로그 스트림)**              |
| **탐지 성격**         | 취약점 및 설정 오류 (Misconfigurations)                | 실시간 보안 위협 및 비정상 행위 (Threats/Attacks)           |
| **권한 회수 시 동작** | **자동으로 비활성화되어 사라짐 (Silently Disappears)** | **수동 해결 전까지 활성 유지 (Stay Active Until Resolved)** |
| **사후 조사(포렌식)** | ❌ 흔적이 지워지므로 조사 불가                         | ⭕ **완벽한 타임라인 및 침해 조사 가능**                    |

---

## 7. 실전 시나리오: 외부 계정 비정상 권한 부여 (Anomalous IAM Grants)

### 7.1 시나리오 개요

- 조직 내 어떤 프로젝트에서든 외부 개인 계정(`@gmail.com`)에 `Owner` 등 강력한 권한이 부여될 때
  실시간 경보를 받아야 함.
- 침해 사고 분석을 위해, **사후에 권한이 회수되더라도 Finding이 조용히 사라지지 않고
  조사(Investigate)할 수 있도록 보존**되어야 함.

### 7.2 정답 아키텍처: ETD + 조직 레벨 활성화

1. **탐지 카테고리**: `Persistence: IAM Anomalous Grant`
   - MITRE ATT&CK 지속성(Persistence) 전술에 기반하여, 외부 공격자나 내부 위협 행위자가 백도어
     권한을 획득하는 행위를 즉시 탐지합니다.
2. **조직 레벨(`at the organization level`) 활성화의 필수성**:
   - **미래 예측 불가**: 수백 개 프로젝트 중 어느 프로젝트에 외부 권한이 부여될지 사전에 알 수
     없습니다.
   - **전사 거버넌스 보장**: 조직 상위에서 단 한 번 활성화하면, 현재의 모든 프로젝트뿐만 아니라
     **미래에 생성될 신규 프로젝트까지 자동으로 보안 감시가 상속**됩니다.

---

## 8. 시험 대비 핵심 암기 공식

| 문제 키워드                                                                      | 정답 연상 패턴                                                            | 오답 함정                                                                      |
| :------------------------------------------------------------------------------- | :------------------------------------------------------------------------ | :----------------------------------------------------------------------------- |
| **"조용히 사라지지 않고 조사 가능해야 함"<br>(rather than silently disappears)** | **Event Threat Detection (ETD)**<br>(stay active until manually resolved) | Security Health Analytics (SHA) ❌<br>(상태 복구 시 자동 비활성화됨)           |
| **"조직 전체를 대상으로"<br>(across your entire organization)**                  | **At the organization level**                                             | At only the specific project level ❌<br>(사고 발생 프로젝트를 사전 예측 불가) |
| **"정기 규정 준수 및 구성 오류 스캔"**                                           | **Security Health Analytics (SHA)**                                       | -                                                                              |
