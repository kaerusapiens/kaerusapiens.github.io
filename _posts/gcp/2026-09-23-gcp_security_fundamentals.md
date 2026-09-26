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

- **개념**: _"네트워크 내부라고 해서 절대 신뢰하지 않는다(Never Trust, Always Verify)."_ 네트워크
  경계 내부/외부를 구분하지 않고 모든 요청에 대해 ID, 디바이스 상태, 컨텍스트를 지속적으로
  검증합니다.
- **핵심 구성요소**:
  - **Identity-Aware Proxy (IAP)**: VPN 없이 HTTPS 기반으로 컨텍스트 인지형 접근 제어 제공.
  - **Context-Aware Access**: 사용자 위치, 기기 보안 상태(OS 버전, 디스크 암호화 여부)에 따른 조건부
    액세스.
