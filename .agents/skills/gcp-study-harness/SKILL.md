---
name: gcp-study-harness
description:
  "사용자가 제공하는 GCP Security 기출문제/학습 내용에서 저작권 요소를 제외하고 핵심 개념만 추출하여
  리소스/목적별 블로그 포스트에 자동 업데이트하는 AI 워크플로우입니다."
---

# GCP Study Harness

## 목적 (Purpose)

사용자가 GCP Professional Cloud Security Engineer 기출문제나 공부한 내용을 입력했을 때, 원본 문제
지문이나 보기(저작권 문제 소지)는 저장소에 남기지 않고 **오직 사용자가 이해해야 할 핵심 개념과 모범
사례만 추출**하여 해당 리소스/목적에 맞는 블로그 포스트에 지속적으로 누적 기록합니다.

## 대상 파일 구조 (`_posts/gcp/`)

주제 및 리소스별로 역할을 분리하여 관리합니다:

1. **시험 안내 및 로드맵**: `_posts/gcp/YYYY-MM-DD-gcp_security_guide.md` (시험 가이드, 출제 범위,
   주차별 체크리스트)
2. **GCP 보안 전반**: `_posts/gcp/YYYY-MM-DD-gcp_security_fundamentals.md` (최소 권한, 심층 방어,
   공유 책임 모델, 제로 트러스트 등 공통 원칙)
3. **리소스별 학습 포스트**:
   - 네트워크 보안: `_posts/gcp/YYYY-MM-DD-gcp_network_security.md` (서브넷, VPC, PGA, Cloud NAT,
     PSC, VPN 등)
   - IAM 및 계정 보안: `_posts/gcp/YYYY-MM-DD-gcp_iam_security.md` (신규 주제 발생 시 생성)
   - 데이터 보호 및 암호화: `_posts/gcp/YYYY-MM-DD-gcp_data_security.md` (KMS, DLP 등)
   - 보안 운영 및 로깅: `_posts/gcp/YYYY-MM-DD-gcp_operations_security.md` (SCC, Cloud Audit Logs
     등)

## AI 에이전트 수행 지침 (Instructions)

사용자가 학습 내용이나 문제를 제공하며 이 스킬의 수행을 요청하면, 다음 단계를 엄격히 따릅니다:

1. **내용 분석 및 정제 (Sanitization & Extraction)**:
   - 제공된 텍스트에서 문제 원문, 특정 보기 내용 등 저작권 침해 소지가 있는 문장은 완벽히
     배제합니다.
   - 문제에서 다루고자 하는 **GCP 서비스의 기능, 아키텍처 원리, 보안 모범 사례, 제약 사항** 등
     "이해한 개념" 위주로 요약합니다.

2. **대상 파일 판별 및 업데이트 (Route & Update)**:
   - 추출된 개념의 도메인(네트워크, IAM, 데이터 보안, 보안 운영 등)을 파악하여 적절한 리소스
     파일(`_posts/gcp/*`)의 하단 섹션에 추가합니다.
   - 아직 해당 리소스 파일이 없다면, 표준 Front Matter 규격을 갖춘 신규 포스트 파일을 생성하여
     배치합니다.
   - 가독성을 위해 H2/H3 제목, 표, Markdown Bullet list(`-`), 주석이 포함된 코드 블록을 사용하여
     정리합니다.

3. **사용자 피드백 (User Feedback)**:
   - 어떤 파일의 어느 섹션에 어떤 개념이 추가되었는지 요약하여 보고합니다.
   - 깃 커밋 및 푸시 필요 여부를 확인합니다.

## 사용 예시 (Usage)

사용자가 채팅창에 기출문제를 복사+붙여넣기 한 뒤, **"하네스로 정리해줘"** 또는 **"개념
추가해줘"**라고 요청하면 이 스킬이 트리거되어 위 지침을 수행합니다.
