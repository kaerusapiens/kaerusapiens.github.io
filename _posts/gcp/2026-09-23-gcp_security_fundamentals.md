---
# 포스트 기본 메타데이터 설정
title: "GCP 보안 전반: 핵심 아키텍처 원칙 및 보안 모델"
# 발행 일시 (타임존 +0900 명시)
date: 2026-09-23 15:15:00 +0900
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

Google Cloud에서 안전하고 신뢰할 수 있는 엔터프라이즈 인프라를 구축하기 위해서는 특정 개별 리소스
설정 이전에 **GCP 전반을 관통하는 보안 철학과 설계 원칙**을 먼저 이해해야 합니다. 이 글에서는 GCP
보안의 4대 핵심 축인 최소 권한, 심층 방어, 공유 책임 모델, 제로 트러스트(BeyondCorp)를 정리합니다.

---

## 2. 핵심 보안 설계 원칙 (Core Security Principles)

### 2.1 최소 권한의 원칙 (Principle of Least Privilege)

- **개념**: 사용자, 그룹, 서비스 계정(Service Account)에 업무 수행에 **정확히 필요한 최소한의 권한만
  부여**하고, 불필요한 광범위 권한(예: `roles/owner`, `roles/editor`) 부여를 엄격히 금지합니다.
- **적용 방안**:
  - 기본(Primitive) 역할 대신 **사전 정의된(Predefined) 역할** 또는 **커스텀(Custom) 역할** 사용.
  - IAM 조건(Conditions)을 적용하여 시간, IP, 리소스 태그 기반의 세부 제어 구현.
  - **IAM Recommender**를 활용하여 장기간 미사용 권한을 주기적으로 자동 회수.

### 2.2 심층 방어 (Defense in Depth)

- **개념**: 단일 보안 장벽에 의존하지 않고, 물리적 보안부터 네트워크, 호스트, 애플리케이션, 데이터에
  이르기까지 **다중 레이어 방어 체계**를 구축합니다.
- **레이어별 방어 예시**:
  - **네트워크 경계**: Cloud Armor WAF, 방화벽 정책, Cloud IDS.
  - **인스턴스/호스트**: Shielded VM, OS Login, 취약점 스캐닝.
  - **데이터 레이어**: Cloud KMS 기반 CMEK 암호화, Sensitive Data Protection(DLP).
  - **운영/모니터링**: Security Command Center(SCC), Cloud Audit Logs.

### 2.3 클라우드 공유 책임 모델 (Shared Responsibility Model)

- **개념**: 클라우드 서비스 모델(IaaS, PaaS, SaaS)에 따라 Google(클라우드 제공자)과 고객 간의 보안
  책임 분계선이 달라집니다.
- **서비스 모델별 책임 비교**:
  - **IaaS (Compute Engine)**: Google은 하드웨어, 물리 데이터센터, 가상화 레이어를 보호. 게스트 OS,
    패치 관리, 방화벽 설정, 네트워크 구성, 데이터 암호화는 **고객의 책임**.
  - **PaaS (Cloud Run, GKE Autopilot, App Engine)**: OS 패치와 런타임 환경은 Google이 관리.
    애플리케이션 코드, IAM 접근 제어, 데이터 보안은 **고객의 책임**.
  - **SaaS (Google Workspace, BigQuery 완전관리형)**: 대부분의 인프라 보안을 Google이 전담. 데이터
    분류, 사용자 계정 및 접근 권한 관리는 **고객의 책임**.

### 2.4 제로 트러스트 (Zero Trust - BeyondCorp)

- **핵심 철학**: _"절대 신뢰하지 말고, 언제나 검증하라 (Never Trust, Always Verify)."_
- **접근 방식**: 네트워크 위치(내부망 vs 외부망)에 의존하지 않고, **사용자 신원(ID) + 기기 보안
  상태(Context)**를 모든 요청마다 지속적으로 검증합니다.
- **주요 구현체**: Identity-Aware Proxy(IAP), Context-Aware Access, VPC Service Controls(VPC SC).

---

## 3. 제로 트러스트(Zero Trust) 심층 분석: BeyondCorp 아키텍처

### 3.1 전통적 경계 보안(성곽 모델)의 한계와 패러다임 전환

과거의 엔터프라이즈 보안은 네트워크 경계(방화벽, VPN)를 굳건히 지키는 **'성곽과
해자(Castle-and-Moat)'** 모델이었습니다. 하지만 재택근무, 모바일 기기, 멀티 클라우드 및 SaaS
도입으로 "내부망"이라는 경계 자체가 붕괴되었습니다.

```text
[ 전통적 보안: 성곽과 해자 (Castle and Moat) ]
   침입자 ──(VPN/방화벽 통과)──▶ [ 내부망 진입 성공! ]
                                   │
                                   ▼ (내부 검사 없음)
                                 DB, 내부 서버, 결제망 자유롭게 유린 (횡적 이동)

----------------------------------------------------------------------

[ 제로 트러스트 (Zero Trust - BeyondCorp) ]
   사용자 ──(매 요청마다 검증)──▶ [ 리소스 A (웹 서버) ]  (인가 성공)
      │
      └───(또 다시 별도 검증)──▶ [ 리소스 B (DB 서버) ]   (권한 없으면 즉시 차단!)
           * 신원(ID), 기기 상태(OS 패치, 암호화), 위치를 매 순간 지속 검증
```

- **성곽 모델의 치명적 약점**: 내부 직원의 PC가 악성코드에 감염되거나 VPN 계정이 유출되면, 내부망
  전체로 침투가 확산되는 **횡적 이동(Lateral Movement)**을 막을 수 없습니다.
- **제로 트러스트의 접근법**: *"내부 사내망이든 외부 공용 와이파이든 똑같이 신뢰할 수 없는 위험한
  환경이다"*라고 전제하고, 모든 개별 리소스 앞단에 검증 관문을 설치합니다.

---

### 3.2 제로 트러스트 3대 핵심 원칙 (NIST SP 800-207)

1. **명시적 검증 (Verify Explicitly)**:
   - 사용자의 IP 주소나 네트워크 위치를 맹신하지 않습니다.
   - 사용자 신원(ID), 다중 인증(MFA), 접속 위치, 기기 보안 상태(OS 버전, 디스크 암호화) 등 **사용
     가능한 모든 컨텍스트 데이터를 바탕으로 매번 인증 및 인가**합니다.
2. **최소 권한 원칙 (Use Least Privilege Access)**:
   - 업무에 꼭 필요한 순간에만 일시적으로 권한을 부여하는 **Just-In-Time (JIT)** 및 필요한 최소
     범위만 부여하는 **Just-Enough-Access (JEA)**를 적용합니다.
3. **침해 가정 (Assume Breach)**:
   - *"네트워크 내부에 이미 해커가 상주하고 있다"*고 가정합니다.
   - 네트워크를 미세하게 분할하는 마이크로 세그멘테이션(Micro-segmentation)을 적용하고, 내부
     트래픽도 종단 간 암호화(End-to-End Encryption)하며, 모든 활동을 실시간 로깅/분석합니다.

---

### 3.3 Google Cloud의 제로 트러스트 핵심 서비스 (BeyondCorp)

Google은 2010년 오로라 작전(Operation Aurora) 공격을 계기로 사내망 VPN을 완전히 폐지하고
**BeyondCorp**라는 독자적인 제로 트러스트 보안 프레임워크를 상용화했습니다.

#### (1) Identity-Aware Proxy (IAP) ⭐⭐⭐

- **개념**: 사설 VPC 내의 VM 인스턴스(SSH, RDP)나 사내 웹 애플리케이션에 접근할 때, **외부 공용 IP나
  VPN 없이 브라우저/HTTPS 기반으로 안전하게 접속**하도록 통제하는 가상 프록시입니다.
- **동작 원리**: 사용자가 리소스에 접근하려 하면 IAP가 먼저 가로채 Google Workspace/Cloud Identity
  계정 인증 및 IAM 역할 권한을 확인한 뒤 세션을 허용합니다.

#### (2) Context-Aware Access (컨텍스트 인지형 접근 제어)

- 사용자 계정 정보뿐만 아니라 접속 환경의 **세부 컨텍스트(Context)**를 조건으로 평가합니다:
  - 회사에서 지급한 암호화된 관리 기기인가?
  - 최신 보안 패치가 적용된 OS인가?
  - 사전에 허용된 특정 국가/IP 대역에서 접속하고 있는가?

#### (3) VPC Service Controls (VPC SC)

- 외부 침입 차단을 넘어, **인증된 내부 사용자에 의한 민감 데이터 유출(Data Exfiltration)을 원천
  차단**하는 제로 트러스트 보안 경계(Service Perimeter)입니다.
- BigQuery, Cloud Storage 등 Google 관리형 API 주변에 방어벽을 둘러, 인가되지 않은 외부 프로젝트로
  데이터를 복사하거나 다운로드하는 행위를 차단합니다.
