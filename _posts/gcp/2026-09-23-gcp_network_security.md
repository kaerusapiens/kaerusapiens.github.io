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

## 4. GCP 라우터 & 게이트웨이 핵심 서비스 5대 분류

트래픽의 **방향(Direction)**과 **목적지(Destination)**에 따라 사용해야 하는 GCP 네트워크 서비스가
명확히 구분됩니다:

- **외부 인터넷으로 나갈 때 (Outbound)**: `Cloud NAT`
- **온프레미스/타 클라우드와 연결할 때 (Hybrid)**: `Cloud Router`, `Cloud VPN (HA VPN)`,
  `Cloud Interconnect`
- **VPC 간 또는 서비스 간 사설 연결 (Private)**: `VPC Peering`, `Private Service Connect (PSC)`
- **외부에서 들어오는 대문 (Inbound)**: `Cloud Load Balancing (Gateway API)`

```text
               [ 온프레미스 / 타 클라우드 ]
                     ▲             ▲
                     │ (전용선)     │ (IPsec 암호화)
             Cloud Interconnect   Cloud VPN (HA VPN)
                     │             │
                     ▼             ▼
   [ VPC 경계 ] ───▶ [ Cloud Router ] (동적 경로 BGP 안내판)
         │
         ├───▶ [ Cloud NAT ] ─────────▶ [ 외부 공용 인터넷 (아웃바운드 전용) ]
         │
         ├───▶ [ Private Service Connect (PSC) ] ─▶ [ Google API / 타사 SaaS 사설 연동 ]
         │
         └───▶ [ VPC Peering ] ───────▶ [ 다른 사설 VPC ]
```

---

### 4.1 네트워크의 두뇌 (동적 라우팅 안내판): Cloud Router

- **역할**: 가상 라우터로서, 온프레미스나 타 클라우드 장비와 **BGP(Border Gateway Protocol)** 통신을
  통해 실시간으로 네트워크 경로 정보를 주고받습니다.
- **핵심 특징**:
  - 실제 패킷 트래픽이 Cloud Router 장비를 통과하는 것이 아니라, **경로 정보(Control Plane)만
    계산하여 가상 머신에 전달**합니다. (따라서 라우터 자체로 인한 대역폭 병목이 없습니다.)
  - `Cloud NAT`, `Cloud VPN`, `Cloud Interconnect`를 구성하기 위한 필수 전제 조건입니다.

---

### 4.2 안전한 아웃바운드 인터넷 게이트웨이: Cloud NAT

- **역할**: 외부 IP가 없는 내부 사설 VM들이 OS 패키지 다운로드(`apt-get`, `yum`), 외부 API 호출을
  위해 **외부 인터넷으로 나갈 수 있게 해주는 단방향 출구**입니다.
- **보안 포인트**:
  - **아웃바운드(Outbound) 전용**: 외부 인터넷에서 내부 VM으로 직접 접속하는 인바운드는 원천
    차단됩니다.
  - 별도의 NAT 프록시 VM 인스턴스를 띄우지 않는 **서버리스(분산) 게이트웨이**이므로 단일
    장애점(SPOF) 없이 고가용성이 보장됩니다.

---

### 4.3 온프레미스 & 하이브리드 연결 게이트웨이

#### (1) Cloud VPN (HA VPN)

- **역할**: 인터넷 공용망을 통해 온프레미스 장비와 GCP VPC 사이에 **IPsec 암호화 터널**을 뚫어주는
  가상 게이트웨이입니다.
- **보안 포인트**:
  - **99.99% 가용성**을 보장하는 HA VPN은 두 개의 독립된 터널 인터페이스(Active/Active 또는
    Active/Passive)를 필수로 구성하도록 강제합니다.

#### (2) Cloud Interconnect

- **역할**: 공용 인터넷을 타지 않고, 통신사 전용선(Direct Fiber)을 통해 기업 데이터센터와 Google
  망을 물리적으로 직접 연결하는 **초고속 전용선 게이트웨이**입니다.
- **보안 포인트**:
  - 데이터가 공용 인터넷에 노출되지 않으며, 대용량 트래픽에 대한 보안성과 네트워크 안정성이 가장
    높습니다.

---

### 4.4 VPC 간 및 서비스 사설 연결 (보안 시험 빈출!)

#### (1) Private Service Connect (PSC) ⭐

- **역할**: 다른 프로젝트의 VPC 서비스, 타사 SaaS, Google API를 **내 서브넷 내부의 사설
  IP(Endpoint)로 직접 끌고 와서 연결**해 주는 차세대 제로 트러스트 연결 기술입니다.
- **보안 포인트**:
  - 기존 VPC Peering과 달리 **IP 대역 충돌(Overlapping IP) 문제가 발생하지 않습니다**.
  - 양쪽 네트워크 전체를 연결하지 않고, 오직 **특정 서비스 단일 IP 엔드포인트만 노출**하므로 제로
    트러스트(최소 권한) 원칙에 완벽히 부합합니다.

#### (2) VPC Network Peering

- **역할**: 서로 다른 두 VPC 네트워크를 연결하여, 양쪽 VPC의 모든 서브넷이 사설 IP로 통신할 수 있게
  해줍니다.
- **주의점**:
  - 게이트웨이나 홉(Hop)이 없어 레이턴시와 비용 면에서 유리하지만, **양쪽 VPC의 IP 대역이 겹치면
    피어링을 생성할 수 없습니다**.

---

### 4.5 서버리스를 위한 내부 진입로: Serverless VPC Access

- **역할**: Cloud Functions, Cloud Run 같은 서버리스 환경은 기본적으로 VPC 외부에 있습니다. 이들이
  VPC 내부의 사설 DB(Cloud SQL 사설 IP 등)로 안전하게 들어갈 수 있도록 통로를 열어주는 커넥터
  게이트웨이입니다.

---

### 4.6 서비스 한눈에 비교하기 (Security 관점)

| 서비스명           | 주요 목적                            | 트래픽 방향               | 외부 인터넷 노출 여부    |
| :----------------- | :----------------------------------- | :------------------------ | :----------------------- |
| **Cloud Router**   | BGP 기반 경로 동적 전파              | 제어 평면 (Control Plane) | ❌ 사설/공용 경로 제어   |
| **Cloud NAT**      | 사설 VM의 인터넷 아웃바운드          | 내부 ➡️ 외부 (단방향)     | ⭕ 외부 공용 IP 경유     |
| **HA VPN**         | 하이브리드 IPsec 암호화 연결         | 양방향 (Site-to-Site)     | ⭕ 인터넷 위 암호화 터널 |
| **Interconnect**   | 전용선 물리 직결                     | 양방향                    | ❌ 완전 전용 사설망      |
| **PSC (Endpoint)** | Google/타사 API 사설 엔드포인트 연결 | 내부 ➡️ 특정 서비스       | ❌ 완전 사설 통신        |

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
