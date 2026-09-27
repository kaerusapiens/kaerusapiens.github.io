---
# 포스트 기본 메타데이터 설정
title: "GCP 데이터 보안: Cloud KMS, Cloud HSM, Cloud EKM 3대 암호화 키 관리"
# 발행 일시 (타임존 +0900 명시)
date: 2026-09-23 16:00:00 +0900
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

Google Cloud는 저장되는 모든 데이터를 기본적으로 암호화(Default Encryption at Rest, Google 소유 및
관리 키)합니다. 하지만 금융 규제(PCI DSS), 의료 컴플라이언스(HIPAA), 데이터 주권(GDPR) 요구사항을
충족하기 위해서는 **기업이 암호화 키를 직접 생성, 회전, 통제할 수 있는 고객 관리 암호화 키(CMEK)**
아키텍처가 필수적입니다.

이 글에서는 Google Cloud의 암호화 키 관리 서비스(KMS) 3대 티어인 **Cloud KMS(소프트웨어)**, **Cloud
HSM(하드웨어)**, **Cloud EKM(외부 키)**의 차이점과 실제 실무 적용 방법을 정리합니다.

---

## 2. 암호화 3대 티어 직관적 비유: "열쇠를 어디에 보관하는가?"

| 티어             | 비유                               | 열쇠가 보관되는 물리적 위치                          | 특징                                                   |
| :--------------- | :--------------------------------- | :--------------------------------------------------- | :----------------------------------------------------- |
| **1. Cloud KMS** | **스마트폰 앱 속 비밀번호**        | Google의 소프트웨어 서버 메모리                      | 저렴하고 빠르며 대부분의 엔터프라이즈 요구 충족        |
| **2. Cloud HSM** | **은행의 방화/방폭 특수 대여금고** | Google 데이터센터 내부의 물리적 HSM 전용 하드웨어 랙 | 규제 준수 (FIPS 140-2 Level 3, 물리 침투 시 칩 자폭)   |
| **3. Cloud EKM** | **우리 집 안방 깊숙한 금고**       | Google 외부 (고객사의 사내 전산실)                   | Google에 열쇠를 절대 주지 않는 궁극의 데이터 주권 통제 |

---

## 3. Cloud KMS와 Cloud HSM의 상관관계 (아키텍처 계층도)

많은 엔지니어들이 처음에 가장 혼란스러워하는 질문:

> _"Cloud HSM을 써야 하는데, 왜 자꾸 Cloud KMS가 언급되고 Cloud KMS API를 호출하는 걸까요?"_

### 3.1 본질: "단일 통합 창구(KMS)와 하드웨어 엔진 옵션(HSM)"

GCP에서 Cloud HSM은 별도의 콘솔이나 독립된 API를 가진 분리된 서비스가 아닙니다.  
**Cloud KMS라는 거대한 단일 키 관리 플랫폼 안에서 선택할 수 있는 '보호 수준(Protection Level, 백엔드
엔진 옵션)'**입니다.

- **비유 (자동차와 엔진 옵션)**:
  - **Cloud KMS**: 자동차 모델 (운전석, 대시보드, 핸들 = 통일된 API)
  - **SOFTWARE / HSM / EKM**: 보닛 아래 장착되는 엔진 옵션 (가솔린 엔진 vs FIPS Level 3 특수 방탄
    하이브리드 엔진 vs 외부 트레일러 엔진)
  - 운전자(개발자)가 가속 페달(API 호출)을 밟는 방법은 동일하지만, 실제 연산이 일어나는 물리적
    주체만 달라집니다.

```text
                     [ 개발자 / 애플리케이션 ]
                                │
                                ▼ (오직 하나의 통일된 API로만 통신)
       ┌─────────────────────────────────────────────────┐
       │             Google Cloud KMS API                │  <-- 단일 통합 접수 창구
       │        (cloudkms.googleapis.com)                │
       └────────────────────────┬────────────────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │ (protectionLevel)   │ (protectionLevel)   │ (protectionLevel)
          ▼                     ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│   SOFTWARE 엔진   │  │  Cloud HSM 장비  │  │  Cloud EKM 게이트웨이│
│ (FIPS 140-2 Lv1) │  │ (FIPS 140-2 Lv3) │  │  (온프레미스 연동) │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

### 3.2 구글이 HSM을 KMS에 통합한 2가지 엔지니어링 이유

1. **코드 재작성이 필요 없는 API 통일성 (Developer Experience)**:
   - 만약 KMS와 HSM이 별개 서비스라면, 회사가 규제 준수를 위해 일반 키에서 HSM 키로 전환할 때
     애플리케이션 코드를 전면 재작성해야 합니다.
   - GCP는 동일한 API(`kms.encrypt`, `kms.generateRandomBytes`)를 유지한 채, 설정
     파라미터(`protectionLevel: HSM`)만 바꾸면 즉시 하드웨어 보안으로 전환됩니다.
2. **단일한 IAM 권한 체계와 감사 로그 (Audit Logs)**:
   - 누가 키를 생성하고 사용했는지 추적하는 `Cloud Audit Logs`와 접근
     권한(`roles/cloudkms.cryptoKeyEncrypterDecrypter`)이 소프트웨어 키와 HSM 키에 100% 동일하게
     적용됩니다.

---

## 4. Cloud KMS 리소스 계층 구조와 CryptoKey 아키텍처

### 4.1 Cloud KMS 리소스 계층 구조 (Key Hierarchy)

Cloud KMS의 구조는 **'폴더와 파일'**의 관계와 같습니다:

```text
[ Google Cloud 프로젝트 (Project) ]
  └── [ 리전 (Location, 예: asia-northeast3) ]
        └── [ 키링 (KeyRing) ] : 키들을 모아두는 바구니 / 디렉토리
              └── [ 크립토키 (CryptoKey) ] : 특정 목적의 논리적 키 객체 (예: db-user-key)
                    ├── Version 1 (과거 버전 - 복호화 전용)
                    ├── Version 2 (과거 버전 - 복호화 전용)
                    └── Version 3 (최신 기본 버전 - 신규 암호화 및 복호화)
```

- **KeyRing (키링)**: 키들을 묶어서 관리하는 논리적 그룹(바구니)입니다.
- **CryptoKey (크립토키)**: 실제로 데이터를 암호화/복호화하기 위해 애플리케이션이 호출하는 명명된 키
  객체(컨테이너)입니다.

### 4.2 CryptoKey와 CryptoKeyVersion: 왜 분리되어 있을까요?

> **"회사의 대표 직인(도장)은 하나지만, 주기적으로 도장의 인영(버전)을 새로 파서 교체하는 것과
> 같습니다."**

만약 키가 단 하나의 고정된 문자열이라면:

- 30일이 지나 키를 바꾸는 순간, 과거의 그 키로 암호화해 둔 수백만 건의 DB 데이터나 파일들을 전부
  복호화해서 새 키로 다시 암호화해야 하는 대재앙이 일어납니다.

하지만 GCP의 `CryptoKey`는 **내부에 여러 개의 '키 버전(CryptoKeyVersion)'**을 품고 있습니다:

- **신규 데이터 암호화**: CryptoKey의 **가장 최신 기본 버전(Primary Version)**을 사용합니다.
- **과거 데이터 복호화**: 데이터가 암호화될 당시 사용된 **과거 버전이 자동으로 매칭**되어
  복호화됩니다.
- **개발자 이점**: 개발자는 애플리케이션 코드에 복잡한 키 교체 로직을 짤 필요 없이, 오직 CryptoKey의
  리소스 이름만 호출하면 됩니다.

### 4.3 CryptoKey의 4대 핵심 설정 속성

| 속성                             | 설명                                    | 실무 설정 예시                                                         |
| :------------------------------- | :-------------------------------------- | :--------------------------------------------------------------------- |
| **목적 (Purpose)**               | 키의 용도                               | 대칭 암호화(`ENCRYPT_DECRYPT`), 비대칭 서명(`ASYMMETRIC_SIGN`), MAC 등 |
| **보호 수준 (Protection Level)** | 키가 존재하는 위치 (3대 티어)           | `SOFTWARE`, `HSM` (FIPS Level 3), `EXTERNAL` (EKM)                     |
| **회전 주기 (Rotation Period)**  | 다음 새 버전을 자동 생성하는 주기       | 30일(`30d`), 90일(`90d`) 등                                            |
| **라벨 (Labels)**                | 키를 분류하고 필터링하기 위한 메타 태그 | `compliance=pci-dss`, `env=prod`, `team=payment`                       |

```bash
# 30일 자동 회전 주기와 규제 라벨이 적용된 대칭 CryptoKey 생성 예시
gcloud kms keys create payment-card-key \
    --keyring=finance-keyring \
    --location=asia-northeast3 \
    --purpose=encryption \
    --protection-level=hsm \
    --rotation-period=30d \
    --next-rotation-time=2026-10-26T00:00:00Z \
    --labels=compliance=pci-dss,env=prod
```

---

## 5. Tier 1: Cloud KMS (소프트웨어 기반)

가장 대중적인 **'디지털 도어락'**입니다. Google 기본 키 대신 우리 회사의 보안 담당자가 키 회전
주기와 접근 권한을 직접 통제하고 싶을 때(CMEK) 90% 이상의 기업이 기본으로 사용합니다.

- **보호 수준 (Protection Level)**: `SOFTWARE` (FIPS 140-2 Level 1 충족)
- **주요 용도**: Cloud Storage 버킷, BigQuery 테이블, Compute Engine 디스크 암호화

### 실무 설정 예시 (Cloud Storage 버킷에 CMEK 적용)

```bash
# 1단계: 키를 담을 '키링(KeyRing)' 생성 (리전 단위)
gcloud kms keyrings create my-software-keyring \
    --location=asia-northeast3

# 2단계: 소프트웨어 보호 수준(SOFTWARE)의 대칭 암호화 키 생성
# --protection-level=software (기본값이며 소프트웨어 수준 보호)
gcloud kms keys create my-app-key \
    --keyring=my-software-keyring \
    --location=asia-northeast3 \
    --purpose=encryption \
    --protection-level=software

# 3단계: Cloud Storage 버킷에 이 키를 기본 암호화 키로 적용 (CMEK)
# 이제 이 버킷에 업로드되는 모든 객체는 my-app-key로 암호화됩니다.
gcloud storage buckets update gs://my-secure-bucket \
    --default-encryption-key=projects/[프로젝트ID]/locations/asia-northeast3/keyRings/my-software-keyring/cryptoKeys/my-app-key
```

---

## 5. Tier 2: Cloud HSM (하드웨어 보안 모듈)

신용카드 결제망(PCI DSS), 암호화폐 자산 관리, 전자서명 키 등 **법률이나 규정상 하드웨어 보호가
의무화된 경우** 사용합니다.

- **보호 수준 (Protection Level)**: `HSM` (**FIPS 140-2 Level 3 인증**)
- **핵심 특징**:
  - Google 데이터센터에 설치된 특수 하드웨어 칩 안에서 키가 생성되며, **평문 키는 하드웨어 칩 밖으로
    절대 추출(Export)되지 않습니다**.
  - **FIPS 140-2 Level 3**: 물리적 분해나 전압 공격 등 물리 침투가 감지되면 칩 내부 데이터가 즉시
    0으로 덮어써져 자폭(Zeroization)됩니다.
  - 암호키 생성 외에도, 물리 칩의 열잡음을 이용해 예측 불가능한 순수 하드웨어 난수를 추출하는
    **`generateRandomBytes` API**를 제공합니다.

> [!TIP] **자격증 시험 빈출 함정 포인트 (Cloud HSM & 난수 생성)**
>
> 1. **`generateRandomBytes` API 크기 제한**: 1회 API 호출 시 **최대 1024바이트(1 KiB)**까지만
>    요청할 수 있습니다. 2048바이트 등을 요청하면 API 에러(`INVALID_ARGUMENT`)가 발생합니다.
> 2. **소프트웨어 난수를 암호화하는 함정**: 로컬 소프트웨어로 생성한 난수를 HSM 키로 암호화한다고
>    해서 컴플라이언스가 요구하는 '하드웨어 난수'가 되지 않습니다. 난수 값 자체가 물리 HSM 칩에서
>    생성되어야 합니다.

### 실무 설정 예시 (HSM 키 생성 및 민감 데이터 암호화)

```bash
# 1단계: HSM 키를 담을 키링 생성
gcloud kms keyrings create my-hsm-keyring \
    --location=asia-northeast3

# 2단계: 보호 수준을 HSM으로 지정하여 하드웨어 키 생성
# FIPS 140-2 Level 3 검증 하드웨어 모듈 내부에서 키가 안전하게 생성됩니다.
gcloud kms keys create my-pci-dss-key \
    --keyring=my-hsm-keyring \
    --location=asia-northeast3 \
    --purpose=encryption \
    --protection-level=hsm

# 3단계: 하드웨어 칩을 이용해 민감한 결제 데이터 파일 직접 암호화
# 암호화 연산 자체가 구글 데이터센터의 물리 HSM 칩 내부에서 수행됩니다.
gcloud kms encrypt \
    --key=my-pci-dss-key \
    --keyring=my-hsm-keyring \
    --location=asia-northeast3 \
    --plaintext-file=card_numbers.txt \
    --ciphertext-file=card_numbers.enc

# (참고) FIPS 140-2 Level 3 하드웨어 칩으로부터 256바이트 순수 난수 직접 추출
gcloud kms generate-random-bytes \
    --location=asia-northeast3 \
    --protection-level=hsm \
    --num-bytes=256 \
    --output-file=session_token_seed.bin
```

---

## 6. Tier 3: Cloud EKM (외부 키 관리자, External Key Manager)

**"Hold Your Own Key (HYOK)"** 모델입니다. 유럽 연합(EU)의 GDPR 규정처럼 **"미국 클라우드 제공업체
서버에 암호키가 아예 존재해서는 안 된다"**는 극단적인 규제를 준수할 때 사용합니다.

```text
[ 고객 사내 전산실 (On-Premises) ]         [ Google Cloud 환경 ]
┌──────────────────────────────┐          ┌──────────────────────────────┐
│  고객사의 타사 HSM 장비       │          │  BigQuery 데이터 분석 쿼리   │
│  (Thales, Fortanix 등)       │          │  SELECT * FROM secret_data;  │
│                              │          └──────────────┬───────────────┘
│  [ 마스터 암호키 보관 ]       │                         │
│             ▲                │                         ▼
│             │ (복호화 요청)   │          ┌──────────────────────────────┐
│             └────────────────┼──────────┤ Cloud EKM (프록시 게이트웨이) │
│               인터넷/전용선  │ (HTTPS)  └──────────────────────────────┘
└──────────────────────────────┘
  * 고객사가 사내 키 서버 전원 코드를 뽑아버리면?
    -> 구글 BigQuery는 즉시 데이터를 1바이트도 읽지 못하고 차단됨 (완벽한 데이터 주권 확보)
```

- **보호 수준 (Protection Level)**: `EXTERNAL` 또는 `EXTERNAL_VPC`
- **핵심 특징**:
  - 암호키가 Google Cloud에 존재하지 않고 **고객 사내 전산실의 타사 HSM 장비(Thales, Fortanix
    등)**에 보관됩니다.
  - Google Cloud 서비스가 데이터를 복호화해야 할 때마다 네트워크를 통해 고객사의 EKM 엔드포인트로
    실시간 승인을 요청합니다.

### 실무 설정 예시 (외부 EKM 키 등록)

```bash
# 외부 사내 데이터센터의 EKM 서버 엔드포인트를 바라보는 EKM 키 등록
# --protection-level=external (구글 외부 시스템과 연동)
gcloud kms keys create my-ekm-key \
    --keyring=my-ekm-keyring \
    --location=asia-northeast3 \
    --purpose=encryption \
    --protection-level=external \
    --external-key-uri="https://ekm.mycompany.com/v1/keys/master-key-01"
```

---

## 7. 한눈에 보는 비교 및 실무 선택 가이드

| 비교 항목             | 1. Cloud KMS                  | 2. Cloud HSM                        | 3. Cloud EKM                               |
| :-------------------- | :---------------------------- | :---------------------------------- | :----------------------------------------- |
| **보호 수준 (Flag)**  | `--protection-level=software` | `--protection-level=hsm`            | `--protection-level=external`              |
| **FIPS 인증 등급**    | FIPS 140-2 Level 1            | **FIPS 140-2 Level 3**              | 고객사 HSM 스펙에 따름                     |
| **키 생성/연산 위치** | Google 클라우드 소프트웨어    | Google 데이터센터 전용 HSM 랙       | **고객사 사내 전산실 (Google 외부)**       |
| **비용**              | 가장 저렴 (키당 월 $0.06)     | 중간 (키당 월 $1.00 이상)           | 외부 HSM 장비 및 라이선스 비용 발생        |
| **주요 사용 대상**    | 90% 이상의 일반 엔터프라이즈  | 금융, 결제(PCI DSS), 인증 세션 토큰 | 유럽 공공기관, 국가 안보, 최고 규제 기업   |
| **레이턴시 영향**     | 거의 없음 (초고속)            | 매우 적음                           | 외부 네트워크 왕복으로 인한 지연 발생 가능 |

---

## 8. BigQuery 동적 데이터 마스킹과 테라폼 CI/CD 보안 아키텍처

사원 계정, PC 접속 비밀번호, 주민등록번호와 같은 고위험 기밀 데이터를 BigQuery에 저장할 때, **컬럼
수준 보안(Column-level Security)** 및 **동적 데이터 마스킹(Dynamic Data Masking)**을 구성하여 무단
열람을 원천 차단할 수 있습니다.

특히 이를 **Terraform(IaC) CI/CD 파이프라인**으로 자동 배포할 때, 서비스 어카운트(Service Account)
권한 관리는 엔터프라이즈 제로 트러스트(Zero Trust)의 핵심 관문입니다.

---

### 8.1 아키텍처 개요 및 동작 원리

```text
[ BigQuery 테이블 (사원 ID / 이름 / PC 임시 비밀번호) ]
                       │
                       ▼
          [ Dataplex Policy Tag 분류 ]
             (예: PII-High-Confidential)
                       │
        ┌──────────────┴──────────────┐
        ▼                             ▼
[ 보안팀 그룹 ]               [ 일반 쿼리 사용자 / 분석가 ]
• roles/datacatalog.          • roles/bigquery.dataViewer
  categoryFineGrainedReader   • Policy Tag 권한 없음
        │                             │
        ▼                             ▼
[ 🔓 원본 비밀번호 조회 ]      [ 🔒 SHA256 해시 / NULL 마스킹 처리 ]
```

1. **Dataplex Taxonomy & Policy Tag**: 민감 데이터 분류 체계(예: Confidential, Secret)를 정의합니다.
2. **BigQuery Dynamic Data Masking Rule**: 태그가 부착된 컬럼을 비인가자가 쿼리할 때 반환할 마스킹
   규칙(`SHA256`, `ALWAYS_NULL`, `DEFAULT_MASKING_VALUE` 등)을 선언합니다.
3. **IAM 기반 접근 제어**:
   - `roles/datacatalog.categoryFineGrainedReader` 권한을 보유한 주체만 마스킹을 해제(Unmask)하고
     원본 데이터를 열람할 수 있습니다.

---

### 8.2 테라폼 CI/CD 파이프라인 권한 분리: 안티패턴 vs 제로 트러스트

테라폼으로 마스킹 인프라와 권한을 배포할 때, **CI/CD 파이프라인 러너(Service Account)의 권한
설계**가 시스템 전체의 보안을 좌우합니다.

```text
[ ❌ 위험한 안티패턴 ]
  테라폼 CI/CD 러너 ──▶ datacatalog.admin (인프라 생성) + categoryFineGrainedReader (언마스크 열람)
                            │
                            ▼
    * CI/CD 계정이 해킹당하면 해커가 테라폼 권한으로 사원 비밀번호를 즉시 탈취(Exfiltration)!
-----------------------------------------------------------------------------------------
[ ⭕ 올바른 제로 트러스트 패턴 ]
  테라폼 CI/CD 러너 ──▶ [ 인프라 생성 권한만 부여 ] (datacatalog.admin, bigquery.admin)
                            │
                            ▼
  실제 언마스크 권한 ──▶ [ 오직 실제 사람(보안팀 그룹)에게만 테라폼 코드로 부여 ]
                         (group:security-team@company.com -> categoryFineGrainedReader)
```

- **안티패턴의 위험성**: 테라폼 러너에 `categoryFineGrainedReader`를 부여하면, CI/CD 스크립트 변조나
  파이프라인 침해 사고 발생 시 해커가 BigQuery 콘솔/API를 통해 사원 비밀번호 원본을 일괄
  유출(Exfiltration)할 수 있습니다.
- **올바른 설계**: CI/CD 서비스 어카운트는 인프라를 프로비저닝하는 **Control Plane
  권한(`datacatalog.admin`, `bigquery.admin`)**만 보유해야 하며, 실제 데이터를 복호화/언마스크하여
  읽는 **Data Plane 권한(`categoryFineGrainedReader`)**은 절대로 가져서는 안 됩니다.

---

### 8.3 테라폼 실무 구성 시 4대 핵심 원칙

1. **Control Plane과 Data Plane의 철저한 분리**:
   - 테라폼 서비스 어카운트는 정책 태그를 생성하고 IAM 바인딩을 추가할 권한만 가집니다.
   - 데이터 열람 권한은 구글 워크스페이스 그룹(`group:security-team@company.com`)에만 매핑합니다.
2. **`_iam_member` 리소스 사용 필수 (`_iam_binding` 금지)**:
   - `google_data_catalog_policy_tag_iam_binding`은 선언되지 않은 기존 사용자 권한을 전체 덮어쓰고
     삭제(Revoke)하는 치명적인 사이드 이펙트가 있습니다.
   - 따라서 개별 멤버를 추가하는 `google_data_catalog_policy_tag_iam_member`를 사용해야 합니다.
3. **리전(Location) 일치성 보장**:
   - Dataplex Taxonomy / Policy Tag의 `location`과 BigQuery 데이터세트의 `location`은 **반드시
     동일한 리전**(예: `asia-northeast3`)이어야 합니다. 크로스 리전 태깅은 지원되지 않습니다.
4. **필수 API 사전 활성화**:
   - `datacatalog.googleapis.com`
   - `bigquerydatapolicy.googleapis.com`

---

### 8.4 완전한 테라폼(Terraform) 구현 예제

```hcl
# ==============================================================================
# 1. 필수 Google Cloud API 활성화
# ==============================================================================
# Dataplex Data Catalog API
resource "google_project_service" "datacatalog" {
  project = var.project_id
  service = "datacatalog.googleapis.com"
  disable_on_destroy = false
}

# BigQuery Data Policy API (동적 데이터 마스킹 필수 API)
resource "google_project_service" "bigquery_datapolicy" {
  project = var.project_id
  service = "bigquerydatapolicy.googleapis.com"
  disable_on_destroy = false
}

# ==============================================================================
# 2. 데이터 분류 체계 (Taxonomy) 및 정책 태그 (Policy Tag) 생성
# ==============================================================================
# 서울 리전(asia-northeast3)에 데이터 분류 카테고리 정의
resource "google_data_catalog_taxonomy" "security_taxonomy" {
  provider     = google-beta
  project      = var.project_id
  region       = "asia-northeast3"
  display_name = "사내_보안_분류_체계"
  description  = "사원 개인정보 및 PC 인증 자산 분류를 위한 Taxonomy"

  # 세분화된 접근 제어(FINE_GRAINED_ACCESS_CONTROL) 활성화
  activated_policy_types = ["FINE_GRAINED_ACCESS_CONTROL"]

  depends_on = [google_project_service.datacatalog]
}

# 민감 비밀번호 컬럼에 부착할 정책 태그 생성
resource "google_data_catalog_policy_tag" "pc_password_tag" {
  provider          = google-beta
  taxonomy          = google_data_catalog_taxonomy.security_taxonomy.id
  display_name      = "PC_Password_Confidential"
  description       = "사원 PC 비밀번호 등 최상위 기밀 자산에 부착"
}

# ==============================================================================
# 3. BigQuery 동적 데이터 마스킹 규칙 선언
# ==============================================================================
# 권한이 없는 사용자가 조회할 경우 원본 대신 SHA256 해시값으로 마스킹되어 반환
resource "google_bigquery_datapolicy_data_policy" "password_masking_policy" {
  provider         = google-beta
  project          = var.project_id
  location         = "asia-northeast3" # Taxonomy 리전과 동일해야 함
  data_policy_id   = "mask_pc_password_sha256"
  policy_tag       = google_data_catalog_policy_tag.pc_password_tag.name
  data_policy_type = "DATA_MASKING_POLICY"

  # 마스킹 표현식: SHA256 해시 변환 (또는 ALWAYS_NULL 등 지정 가능)
  data_masking_policy {
    predefined_datamask_type = "SHA256"
  }

  depends_on = [google_project_service.bigquery_datapolicy]
}

# ==============================================================================
# 4. 제로 트러스트 IAM 권한 부여 (실제 사람/그룹에게만 언마스크 권한 부여)
# ==============================================================================
# 테라폼 실행 서비스 어카운트가 아닌, 검증된 보안 운영팀 그룹에게만 언마스크(Fine-Grained Reader) 권한 할당
# _iam_binding 대신 _iam_member를 사용하여 기존 권한 유실 방지
resource "google_data_catalog_policy_tag_iam_member" "security_team_unmask" {
  provider   = google-beta
  policy_tag = google_data_catalog_policy_tag.pc_password_tag.name
  role       = "roles/datacatalog.categoryFineGrainedReader"

  # Google Workspace 보안팀 그룹
  member     = "group:security-team@company.com"
}
```
