---
name: gcp-study-harness
description:
  "사용자가 제공하는 GCP Security 기출문제/학습 내용에서 저작권 요소를 제외하고 핵심 개념만 추출하여
  블로그 포스트에 자동 업데이트하는 AI 워크플로우입니다."
---

# GCP Study Harness

## 목적 (Purpose)

사용자가 GCP Professional Cloud Security Engineer 기출문제나 공부한 내용을 입력했을 때, 원본 문제
지문이나 보기(저작권 문제 소지)는 저장소에 남기지 않고 **오직 사용자가 이해해야 할 핵심 개념과 모범
사례만 추출**하여 지정된 블로그 포스트에 지속적으로 누적 기록합니다.

## 대상 파일 (Target File)

- `_posts/gcp/2026-09-23-gcp_cloud_security_engineer.md`

## AI 에이전트 수행 지침 (Instructions)

사용자가 학습 내용이나 문제를 제공하며 이 스킬의 수행을 요청하면, 다음 단계를 엄격히 따릅니다:

1. **내용 분석 및 정제 (Sanitization & Extraction)**:
   - 제공된 텍스트에서 문제 원문, 특정 보기 내용 등 저작권 침해 소지가 있는 문장은 완벽히
     배제합니다.
   - 문제에서 다루고자 하는 **GCP 서비스의 기능, 아키텍처 원리, 보안 모범 사례, 제약 사항** 등
     "이해한 개념" 위주로 요약합니다.

2. **대상 파일 업데이트 (Update File)**:
   - 대상 파일의 `## 5. 핵심 개념 및 기출 요약 (Concept Archive)` 섹션 하단에 추출된 내용을
     추가합니다. (해당 섹션이 없다면 파일 끝에 생성합니다.)
   - 가독성을 위해 H3(`###`) 제목이나 Markdown Bullet list(`-`)를 사용하여 깔끔하게 정리합니다.

3. **사용자 피드백 (User Feedback)**:
   - 텍스트 요약본이 파일에 어떻게 추가되었는지 짧게 요약하여 답변합니다.
   - 깃 커밋(Git Commit)이 필요한지 묻고, 승인 시 `feat: add study concepts` 등의 메시지로 자동
     커밋을 수행합니다.

## 사용 예시 (Usage)

사용자가 채팅창에 기출문제를 복사+붙여넣기 한 뒤, **"gcp-study-harness로 개념 추가해줘"**라고
요청하면 이 스킬이 트리거되어 위 지침을 수행합니다.
