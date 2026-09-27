---
# 포스트 기본 메타데이터 설정
title: "GCP 보안 용어집: 클라우드 보안 필수 용어 사전 (A to Z)"
# 발행 일시 (타임존 +0900 명시)
date: 2026-09-23 16:30:00 +0900
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

Google Cloud Platform(GCP) 및 엔터프라이즈 클라우드 보안 환경에서 자주 등장하는 핵심 보안 전문
용어들을 정리하고 지속적으로 누적하는 사전형 아카이브입니다.

---

## 2. 보안 핵심 용어 사전 (Glossary)

### E

#### Exfiltration (데이터 유출 / 반출) ⭐

- **정의**: 컴퓨터나 조직의 네트워크 내부에서 인가되지 않은 방식으로 중요하고 민감한 데이터(기밀
  문서, 고객 개인정보, 지적 재산 등)를 몰래 빼내 외부로 전송/유출하는 사이버 범죄 행위(Data
  Exfiltration)를 뜻합니다.
- **클라우드 환경의 주요 공격 시나리오**:
  - 인가된 직원이 악의를 품고 내부 BigQuery 데이터를 개인 소유의 외부 프로젝트 버킷으로 복사.
  - 탈취된 서비스 계정 키를 이용해 Cloud Storage 객체를 외부 인터넷 서버로 전송.
  - DNS 터널링(DNS Tunneling)을 통한 소량 분할 데이터 외부 전송.
- **Google Cloud의 주요 방어 대책**:
  - **VPC Service Controls (VPC SC)**: BigQuery, Cloud Storage 등 관리형 API 주변에 강력한 보안
    경계(Service Perimeter)를 설정하여, 인증된 사용자라 할지라도 인가되지 않은 외부 프로젝트로
    데이터를 전송하는 행위를 원천 차단합니다.
  - **Sensitive Data Protection (Cloud DLP)**: 데이터 내 민감 정보(주민등록번호, 카드번호)를 사전
    마스킹 및 암호화하여 유출되더라도 식별 불가하게 조치.
  - **Event Threat Detection (ETD)**: 대량 데이터 다운로드 및 비정상적인 데이터 전송 행위를 실시간
    감사 로그 분석으로 탐지.

---

### C

#### CEL Predicate (Common Expression Language 서술어 / 조건식) ⭐

- **정의**: Google이 개발한 오픈소스 표현식 언어인 **CEL(Common Expression Language)** 환경에서 특정
  조건을 평가하여 **참(true) 또는 거짓(false)**의 불리언(Boolean) 값을 반환하는 논리
  조건식(함수)입니다.
- **컴퓨터 과학적 배경**: 'Predicate(서술어/조건자)'는 값을 입력받아 불리언을 반환하는 함수를
  의미하며, CEL 내에서는 주로 리스트나 맵 같은 컬렉션 데이터를 다루는 매크로(Macros, 예: `all()`,
  `exists()`)와 함께 사용됩니다.
- **GCP 보안에서의 주요 활용처**:
  - **Security Command Center (SCC) Custom Module**: 리소스의 속성(라벨, 회전 주기 등)이 보안 정책을
    위반했는지(`true`/`false`) 실시간 판정.
  - **Cloud IAM Conditions**: 접속 시간, 사용자 IP, 리소스 태그 조건에 따라 권한 부여 여부를
    동적으로 판정.
  - **Organization Policy Custom Constraints**: 생성/수정 요청된 리소스의 속성을 평가하여 배포
    허용(`ALLOW`) 또는 차단(`DENY`) 판정.

#### CDE (Cardholder Data Environment, 카드 소유자 데이터 환경)

- **정의**: 신용카드 번호(PAN), 유효기간, CVV 등 결제 카드 데이터가 단 1바이트라도 저장, 처리,
  전송되는 모든 시스템(서버, 네트워크, DB)의 물리적/논리적 영역.
- **보안 포인트**: PCI DSS 규정 준수를 위해 일반 사내망 및 일반 웹 환경과 프로젝트/네트워크 단위로
  완벽히 격리(Isolation)해야 합니다.

#### CMEK (Customer-Managed Encryption Keys, 고객 관리 암호화 키)

- **정의**: 클라우드 기본 암호화 대신, 고객사 보안팀이 직접 생성, 회전 주기 설정, 권한 제어, 파기를
  관리하는 암호화 키(Cloud KMS).

#### CryptoKey (크립토키)

- **정의**: Cloud KMS에서 생성하는 명명된 암호화 키 객체(컨테이너)로, 내부에 여러 개의 키
  버전(`CryptoKeyVersion`)을 관리하여 무중단 자동 키 회전(Key Rotation)을 가능하게 합니다.

---

### F

#### FIPS 140-2 Level 3

- **정의**: 미국 국립표준기술연구소(NIST)의 하드웨어 보안 모듈 인증 표준.
- **핵심 특징**: 물리적 장비 분해, 전압/온도 공격 등 물리적 침투가 감지되면 칩 내부의 암호키
  데이터를 즉시 0으로 덮어써서 스스로 파괴(Zeroization)하는 고도의 물리 보안 등급입니다. (GCP의
  **Cloud HSM**이 이를 충족합니다.)

---

### J

#### Jailbreak Attempts (탈옥 시도)

- **AI/LLM 보안**: 대형 언어 모델의 안전 가드레일(Safety Alignment) 및 시스템 프롬프트 제약을
  우회(Bypass)하여 금지된 응답 생성을 유도하는 프롬프트 인젝션 공격. (방어: **Model Armor**)
- **전통적 모바일 보안**: OS 커널 취약점을 공격하여 샌드박스를 해제하고 최고 관리자(Root) 권한을
  강제 획득하려는 시도.

---

### P

#### Private Google Access (PGA, 비공개 Google 액세스)

- **정의**: 공용 외부 IP가 없는 사설 VM이 Google 내부 백본망을 통해 Cloud Storage, BigQuery 등
  Google API 서비스에 비공개로 접근할 수 있게 해주는 **서브넷 레벨** 기능.

#### Private Service Connect (PSC)

- **정의**: 다른 VPC의 서비스, Google API, 서드파티 SaaS를 내 서브넷 내부의 사설 IP(엔드포인트)로
  끌고 와 연결하는 차세대 제로 트러스트 연결 기술 (IP 대역 중복 문제 없음).

#### PRNG (Pseudo-Random Number Generator, 소프트웨어 난수 / 의사 난수) ⭐

- **정의**: 컴퓨터 프로그램 코드와 수학 공식으로 계산해 낸 **'가짜 난수(의사 난수)'**입니다.
- **보안 위험성**:
  - 시작 값(Seed)이나 시간 값을 알면 공격자가 **다음에 나올 난수를 완벽히 역산/예측**할 수 있습니다.
  - 만약 인증 세션 토큰을 소프트웨어 난수로 만들면, 해커가 토큰 패턴을 추측하여 다른 사용자의 세션을
    탈취(Session Hijacking)할 위험이 생깁니다.
- **GCP 매핑**: Cloud KMS의 소프트웨어 보호 수준(`protectionLevel: SOFTWARE`, FIPS 140-2 Level 1).

---

### R

#### Redact / Redacted (마스킹 / 비식별화 가림 처리) ⭐

- **정의**: 문서, 보고서, 이미지 등에서 기밀, 개인정보(PII), 민감한 내용 등을 보안이나 법적 이유로
  검열하여 **삭제하거나 검은색 박스 등으로 가려(블라인드/마스킹 처리) 놓은 상태**를 뜻합니다.
- **어원 및 배경**: 원래 동사 형태인 'redact'는 '(원고 등을) 수정하다, 편집하다'라는 뜻이지만,
  현대에는 주로 공공 문서나 법적 서류에서 민감한 정보를 지우는 행위를 가리킵니다.
- **GCP 보안에서의 대표적 구현체**:
  - **Sensitive Data Protection (Cloud DLP)의 `image.redact` API**:
    - 스캔된 이미지 원본을 전송하면 별도의 Vision OCR 파이프라인 없이도, DLP 내부 자체 OCR로
      텍스트를 읽고 `PERSON_NAME`, `EMAIL_ADDRESS`, `PHONE_NUMBER` 등의 민감 영역을 검은색 박스로
      덧칠(Redact)한 이미지를 단 1번의 호출(One Call)로 반환합니다.

---

### T

#### TRNG (True-Random Number Generator, 하드웨어 난수 / 순수 난수) ⭐

- **정의**: 수학 공식이 아닌, **전용 하드웨어 칩 내부의 미세한 물리적 현상(열잡음, 전파 노이즈 등)을
  측정**하여 생성하는 **'진짜 난수(물리적 난수)'**입니다.
- **보안 강점**:
  - 수학 공식이 아니므로 **지구상 어떤 슈퍼컴퓨터로도 다음 값을 예측할 수 없습니다.**
  - 세션 토큰 시드, 암호화 키 생성, 금융 결제망(PCI DSS) 등 최고 수준의 예측 불가능성이 필요할 때
    필수적입니다.
- **GCP 매핑**:
  - **Cloud HSM** (`protectionLevel: HSM`, FIPS 140-2 Level 3 인증 물리 칩).
  - Cloud KMS의 **`generateRandomBytes` API**를 호출하여 물리 HSM 칩에서 직접 순수 난수(1회 호출당
    최대 1024바이트)를 추출할 수 있습니다.

---

### Z

#### Zero Trust (BeyondCorp)

- **정의**: *"절대 신뢰하지 말고, 언제나 검증하라(Never Trust, Always Verify)"*를 원칙으로, 사내
  네트워크 내부라도 신뢰하지 않고 모든 접근 요청에 대해 신원(ID)과 기기 보안 상태(Context)를
  지속적으로 검증하는 보안 패러다임. (구현체: **IAP**, **Context-Aware Access**)
