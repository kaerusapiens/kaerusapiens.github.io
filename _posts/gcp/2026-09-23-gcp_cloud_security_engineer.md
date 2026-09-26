---
# 포스트 기본 메타데이터 설정
title: "GCP Professional Cloud Security Engineer"
# 발행 일시 (타임존 +0900 명시)
date: 2026-09-23 15:30:00 +0900
# 카테고리: 대분류(GCP)와 소분류(Security)로 구조화
categories: [GCP, Security]
# 수식 렌더링 여부
math: false
# 메인 상단 고정 여부
pin: false
# 목차(Table of Contents) 활성화
toc: true
---

## 1. 자격증 개요 (Certification Overview)

**Google Cloud Professional Cloud Security Engineer** 자격증은 보안 모범 사례 및 업계 규정 준수
요구사항을 충족하는 안전한 워크로드와 인프라를 Google Cloud에 설계, 구현, 관리하는 역량을 검증하는
전문 자격증입니다.

- **시험 시간**: 2시간 (120분)
- **문제 형식**: 객관식 및 다중 선택형 (약 50~60문항)
- **시험 언어**: 영어, 일본어 등
- **응시 방식**: 온라인 감독관 시험 또는 공인 테스트 센터 오프라인 응시
- **핵심 역할**:
  - 보안 조직 구조 및 IAM 정책 수립
  - VPC 및 하이브리드 클라우드 네트워크 보안 아키텍처 구성
  - 데이터 암호화(Cloud KMS) 및 데이터 손실 방지(Sensitive Data Protection)
  - 로깅, 모니터링, 취약점 탐지(Security Command Center)를 통한 위협 대응

---

## 2. 시험 출제 영역 (Exam Guide)

GCP 공식 시험 가이드에 기반한 5대 핵심 영역입니다.

```text
├── Section 1: 클라우드 솔루션 환경 내 액세스 구성 (Configuring access) (~23%)
├── Section 2: 네트워크 보안 구성 (Configuring network security) (~21%)
├── Section 3: 데이터 보호 보장 (Ensuring data protection) (~23%)
├── Section 4: 클라우드 환경 내 운영 보안 관리 (Managing operations) (~18%)
└── Section 5: 규정 준수 보장 (Ensuring compliance) (~15%)
```

### Section 1: 클라우드 솔루션 환경 내 액세스 구성

- **Cloud IAM (Identity and Access Management)**:
  - 역할 유형 (기본 역할, 사전 정의된 역할, 커스텀 역할)
  - IAM 조건(Conditions) 및 태그(Tags)를 활용한 조건부 접근 제어
  - 최소 권한 원칙(Principle of Least Privilege) 적용
- **ID 관리 및 연동**:
  - Cloud Identity, Google Workspace 연동
  - 온프레미스 AD/LDAP와의 Google Cloud Directory Sync (GCDS)
  - SAML 2.0 / OIDC 기반 Single Sign-On (SSO)
- **서비스 계정(Service Accounts)**:
  - 서비스 계정 키 생성 방지 및 관리 모범 사례
  - Workload Identity Federation (AWS, Azure, OIDC 제공업체와 연동)
  - GKE Workload Identity
- **조직 정책 (Organization Policies)**:
  - 리소스 계층 구조(조직 > 폴더 > 프로젝트 > 리소스) 상속 규칙
  - 제약 조건(Constraints)을 통한 보안 가드레일 설정 (예: 공용 IP 생성 차단, 특정 리전으로 배포
    제한)

### Section 2: 네트워크 보안 구성

- **VPC 및 네트워크 세분화**:
  - Shared VPC 및 VPC Network Peering 설계 시 보안 고려사항
  - 비공개 Google 액세스 (Private Google Access) 및 Private Service Connect (PSC)
  - 방화벽 규칙 (Ingress/Egress, 방화벽 정책, 계층형 방화벽)
- **경계 보안 및 DDoS 방어**:
  - Cloud Armor (WAF 정책, Rate Limiting, OWASP Top 10 방어, Adaptive Protection)
  - Cloud NAT를 통한 인바운드 차단 및 아웃바운드 인터넷 접근 통제
  - Cloud Load Balancing과 SSL/TLS 정책 (최신 암호화 스위트 적용)
- **위협 탐지 및 인스펙션**:
  - Cloud IDS (Intrusion Detection System)
  - 패킷 미러링 (Packet Mirroring)을 통한 트래픽 검사

### Section 3: 데이터 보호 보장

- **암호화 관리 (Cloud KMS)**:
  - Google 기본 암호화 (Default Encryption at Rest)
  - 고객 관리 암호화 키 (CMEK: Customer-Managed Encryption Keys)
  - 고객 제공 암호화 키 (CSEK: Customer-Supplied Encryption Keys)
  - Cloud HSM (Hardware Security Module) 및 외부 키 관리자(External Key Manager, Cloud EKM)
  - 키 순환(Key Rotation) 정책
- **민감 데이터 보호 (Sensitive Data Protection / Cloud DLP)**:
  - PII (개인정보), 카드번호, 주민등록번호 등 민감 데이터 자동 탐지 및 분류
  - 데이터 마스킹, 토큰화(Tokenization), 암호화 가명화 기법
- **스토리지 및 데이터베이스 보안**:
  - Cloud Storage: Uniform Bucket-Level Access, 버킷 잠금(Bucket Lock), 보존 정책
  - BigQuery: 컬럼 수준 보안(Column-level access control), 행 단위 보안(Row-level security)

### Section 4: 클라우드 환경 내 운영 보안 관리

- **보안 가시성 및 위협 감지**:
  - Security Command Center (SCC) Standard vs Premium
  - 취약점 스캐너 (Web Security Scanner, Container Threat Detection, Event Threat Detection)
  - Anomaly Detection (비정상 탐지)
- **중앙 집중식 로깅 및 모니터링**:
  - Cloud Logging: 감사 로그 (Cloud Audit Logs - Admin Activity, Data Access, System Event, Policy
    Denied)
  - 로그 싱크(Log Sink)를 통한 BigQuery, Cloud Storage, Pub/Sub 내보내기
  - SIEM 연동 (Chronicle, Splunk 등)
- **CI/CD 및 컨테이너 보안**:
  - Artifact Registry 취약점 스캐닝
  - Binary Authorization을 통한 신뢰할 수 있는 컨테이너 이미지만 배포 보장

### Section 5: 규정 준수 보장

- **컴플라이언스 표준 및 프레임워크**:
  - CIS Benchmarks, ISO/IEC 27001, SOC 1/2/3, PCI-DSS, HIPAA, GDPR
- **규정 준수 증적 및 감사**:
  - Compliance Reports Manager 활용
  - 감사 로그 분석 및 불변 보존을 통한 규정 준수 입증

---

## 3. 핵심 보안 설계 원칙 (Core Security Principles)

1. **최소 권한의 원칙 (Least Privilege)**: 사용자 및 서비스 계정에 업무 수행에 필요한 최소한의
   권한만 부여하고 주기적인 IAM 권한 재검토 및 Recommender 활용.
2. **심층 방어 (Defense in Depth)**: 네트워크 경계, 호스트, 애플리케이션, 데이터 레이어 각각에 다층
   방어 기법 구축.
3. **공유 책임 모델 (Shared Responsibility Model)**: IaaS, PaaS, SaaS 서비스 모델에 따른 Google과
   고객 간의 보안 책임 분계선 명확화.
4. **제로 트러스트 (Zero Trust - BeyondCorp)**: 네트워크 내부/외부를 구분하지 않고 모든 요청에 대해
   ID, 디바이스 상태, 컨텍스트를 지속적으로 검증.

---

## 4. 학습 로드맵 및 축적 계획 (Study Checklist)

앞으로 공부하면서 각 영역별 세부 실습과 개념을 지속적으로 기록하고 업데이트할 예정입니다.

- [ ] **1주차: IAM & 리소스 계층 구조 마스터**
  - Cloud Identity / Directory Sync
  - IAM 조건, 커스텀 역할, 태그
  - Service Account 모범 사례 및 Workload Identity
  - Organization Policy 설정 및 테스트
- [ ] **2주차: 네트워크 보안 아키텍처**
  - VPC 방화벽 정책 및 계층형 방화벽
  - Private Service Connect & Cloud NAT
  - Cloud Armor WAF 룰셋 및 Cloud IDS
- [ ] **3주차: 데이터 보안 & Cloud KMS**
  - Cloud KMS 키 계층 구조 (KEK, DEK) 및 CMEK 적용
  - Sensitive Data Protection (DLP) 프로파일링 및 마스킹 실습
  - Cloud Storage / BigQuery 세분화 보안 제어
- [ ] **4주차: 보안 운영, SCC & 위협 탐지**
  - Security Command Center Premium 기능 및 탐지 규칙
  - Cloud Audit Logs 분석 및 BigQuery 기반 보안 분석
  - Binary Authorization 파이프라인 구성
- [ ] **5주차: 모의고사 풀이 및 취약 개념 집중 보완**
  - 공식 샘플 문제 풀이
  - 시나리오 기반 케이스 스터디 및 아키텍처 리뷰

## 5. 핵심 개념 및 기출 요약 (Concept Archive)

이 섹션은 기출문제 및 실습을 통해 학습한 핵심 개념과 모범 사례를 누적하여 기록하는 공간입니다.
