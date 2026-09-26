---
# 포스트 기본 메타데이터 설정
title: "GCP 네트워크 보안: 서브넷, 게이트웨이 및 안전한 통신 (PGA, NAT, PSC)"
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

## 1. 개요 (Overview)

GCP의 Virtual Private Cloud(VPC)는 전 세계 리전을 아우르는 글로벌 소프트웨어 정의
네트워크(SDN)입니다. 안전한 클라우드 아키텍처 구축을 위해 서브넷의 기본 분리 원리부터 외부 인터넷
차단, 비공개 Google API 연동, 하이브리드 연결까지 네트워크 핵심 구성 요소를 정리합니다.

---

## 2. 서브넷(Subnet)의 기본 원리와 네트워크 구성 요소

### 2.1 서브넷(Subnet)의 정의

- 거대한 네트워크(VPC)를 관리, 성능, 보안 목적에 따라 작게 분할한 하위 네트워크 단위입니다.
- GCP에서 VPC는 글로벌 리소스이지만, **서브넷은 특정 리전(Region)에 종속**됩니다.

### 2.2 핵심 네트워크 구성 요소 (비유를 통한 이해)

- **호스트 (Host / VM)**: 통신을 수행하는 엔드포인트 (아파트 입주민).
- **IP 주소 (IP Address)**: 각 호스트에 부여된 고유 식별 번호 (주민의 상세 주소).
- **서브넷 마스크 & CIDR**: IP 주소에서 네트워크 영역(동 번호)과 호스트 영역(호수)을 구분하는
  기준선.
- **기본 게이트웨이 (Default Gateway / Router)**: 다른 서브넷이나 외부 인터넷으로 나갈 때 반드시
  거쳐야 하는 출입구 (동네 경비실).

### 2.3 통신 흐름 원리

- **동일 서브넷 내부 통신**: 게이트웨이를 거치지 않고 가상 스위칭(L2)을 통해 직접 통신합니다.
- **타 서브넷 및 외부 통신**: 서브넷 경계를 넘어야 하므로 기본 게이트웨이(라우터)를 통해 라우팅
  테이블 기반으로 전달됩니다.
- **서브넷 분리의 보안적 이유**:
  - 외부 트래픽을 받는 공개 영역(Public 서브넷)과 중요 데이터를 보관하는 비공개 영역(Private
    서브넷)을 물리적/논리적으로 격리하여 침투 확산(Lateral Movement)을 방어합니다.

---

## 3. Private Google Access (PGA) vs Cloud NAT

외부 공용 IP가 없는 사설 VM이 외부와 통신할 때 적용하는 두 가지 핵심 네트워크 패턴입니다.

### 3.1 Private Google Access (비공개 Google 액세스, PGA)

- **개념**: 외부 IP가 없는 Compute Engine VM이 Google 내부 백본망을 통해 Google Cloud 서비스(Cloud
  Storage, BigQuery 등)에 안전하게 비공개로 접근하도록 지원하는 기능입니다.
- **설정 단위**: **서브넷(Subnet) 레벨**에서 활성화합니다. (VM 개별 설정 불가)
- **보안 이점**:
  - 트래픽이 공용 인터넷으로 나가지 않습니다.
  - 일반 외부 인터넷(공용 웹사이트, 외부 패키지 저장소)으로의 접근 권한은 일절 부여되지 않으므로
    최소 노출 원칙을 충족합니다.

### 3.2 Cloud NAT (Network Address Translation)

- **개념**: 사설 VM이 외부 인터넷(패키지 저장소 `apt-get`, 외부 서드파티 API)으로 나갈 수 있도록
  지원하는 완전 관리형 아웃바운드 게이트웨이입니다.
- **보안 이점**:
  - **단방향(Outbound Only)**: 내부에서 나가는 트래픽에 대한 응답만 허용하며, 외부 인터넷에서 내부
    VM으로의 직접 인바운드 접근은 원천 차단됩니다.

### 3.3 핵심 비교 요약

| 구분                 | Private Google Access (PGA)                       | Cloud NAT                                 |
| :------------------- | :------------------------------------------------ | :---------------------------------------- |
| **주요 목적**        | Google API 및 서비스 비공개 통신                  | 일반 외부 인터넷 아웃바운드 통신          |
| **트래픽 경로**      | Google 내부 백본망 경유                           | NAT 게이트웨이 공용 IP를 통한 인터넷 경유 |
| **외부 인터넷 접근** | ❌ 불가 (Google API 전용)                         | ⭕ 가능 (외부 임의 인터넷 목적지)         |
| **설정 대상**        | 서브넷 속성 (`--enable-private-ip-google-access`) | Cloud Router 기반의 Cloud NAT 게이트웨이  |

---

## 4. GCP 라우터 & 게이트웨이 서비스 체계

GCP의 네트워크 장비는 물리 장비가 아닌 소프트웨어 정의 네트워크(SDN, Andromeda 기반)로 동작하여 단일
장애점(SPOF) 없이 수평 확장됩니다.

1. **Cloud Router**:
   - 온프레미스 장비나 타 클라우드와 BGP(Border Gateway Protocol)를 통해 동적으로 라우팅 정보를
     교환하는 제어 평면(Control Plane) 가상 라우터입니다.
   - 실제 데이터 패킷이 병목을 겪지 않도록 경로 정보만 제공하며, Cloud NAT, Cloud VPN, Cloud
     Interconnect의 기반이 됩니다.
2. **Cloud VPN (HA VPN)**:
   - 공용 인터넷을 통과하는 IPsec 암호화 터널을 제공하며, 99.99% 가용성을 보장하는 고가용성 사이트
     간(Site-to-Site) VPN 게이트웨이입니다.
3. **Cloud Interconnect**:
   - 데이터센터와 Google 망을 통신사 전용선(Direct Fiber)으로 물리 직결하여 대용량 저지연 트래픽을
     공용 인터넷 노출 없이 전송하는 하이브리드 연결 방식입니다.
4. **Private Service Connect (PSC)**:
   - 다른 VPC의 서비스나 타사 SaaS, Google API를 내 서브넷 내부의 사설 IP(Endpoint)로 직접 연결하는
     차세대 제로 트러스트 연결 방식입니다.
   - VPC Peering과 달리 IP 대역 중복 문제가 없으며, 서비스 단위로만 최소 권한 노출이 가능합니다.
5. **Serverless VPC Access**:
   - Cloud Functions, Cloud Run 같은 서버리스 서비스가 VPC 내부의 사설 리소스(Cloud SQL 등)로 진입할
     수 있도록 경로를 열어주는 내부 게이트웨이 커넥터입니다.

---

## 5. 실무 명령어 예시 (gcloud)

```bash
# [1] 서브넷 레벨에서 Private Google Access (PGA) 활성화
# 지정된 리전의 서브넷에 PGA를 켜서 사설 VM이 Google API로 내부 접근 가능하도록 설정합니다.
gcloud compute networks subnets update [서브넷_이름] \
    --region=[리전_명] \
    --enable-private-ip-google-access

# [2] Cloud Router 생성 (BGP 동적 라우팅 안내판 역할)
gcloud compute routers create my-cloud-router \
    --network=[VPC_이름] \
    --region=[리전_명]

# [3] Cloud Router 기반의 Cloud NAT 게이트웨이 생성
# auto-allocate-nat-ips: 공용 IP를 Google이 자동 프로비저닝 및 관리
# nat-all-subnet-ip-ranges: 해당 리전의 모든 서브넷 사설 VM에 아웃바운드 인터넷 허용
gcloud compute routers nats create my-cloud-nat \
    --router=my-cloud-router \
    --region=[리전_명] \
    --auto-allocate-nat-ips \
    --nat-all-subnet-ip-ranges
```
