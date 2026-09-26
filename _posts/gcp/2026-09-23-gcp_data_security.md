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

## 4. Tier 1: Cloud KMS (소프트웨어 기반)

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
