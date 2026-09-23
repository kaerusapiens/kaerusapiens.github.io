# AGENTS.md - Repository Guidelines for AI Agents

이 문서는 이 저장소에서 작업하는 모든 AI 에이전트(Antigravity, Claude Code, GitHub Copilot, Cursor
등)를 위한 프로젝트 가이드라인 및 핵심 제약 사항입니다. 작업을 시작하기 전 반드시 본 문서를 숙지해야
합니다.

---

## 1. Project Overview & Tech Stack

- **Purpose**: [kaerusapiens.github.io](https://kaerusapiens.github.io) 개인 기술 블로그 및
  포트폴리오
- **Base Engine**: Jekyll (Ruby 3.3+)
- **Theme**: [`jekyll-theme-chirpy`](https://github.com/cotes2020/jekyll-theme-chirpy) (Chirpy
  Starter 기반)
- **Deployment Platform**: GitHub Pages (Custom GitHub Actions Workflow)
- **Validation**: HTMLProofer (내부 링크 및 HTML 무결성 검증)

---

## 2. ⚠️ Critical Deployment Constraints (배포 핵심 제약)

> [!CAUTION] **GitHub Pages 배포 소스 방식 엄격 준수 (404 방지)**
>
> 1. **배포 소스 필수 설정**:
>    - 저장소 설정(`Settings > Pages > Build and deployment > Source`)은 반드시
>      **`GitHub Actions`**로 지정되어 있어야 합니다.
>    - **절대로 `Deploy from a branch` (레거시 Jekyll 빌더)로 변경해서는 안 됩니다.**
> 2. **레거시 빌더 덮어쓰기 문제 배경**:
>    - Chirpy 테마는 커스텀 루비 플러그인(`_plugins`) 및 복합 빌드 과정을 필요로 하므로, GitHub 기본
>      Jekyll 빌더에서는 정상 빌드되지 않습니다.
>    - 배포 소스가 `Deploy from a branch`로 설정되면:
>      - 커스텀 워크플로우(`.github/workflows/pages-deploy.yml`)가 정상 빌드 후 배포하더라도,
>      - 약 25~30초 뒤 GitHub 기본 빌더(`pages build and deployment`)가 트리거되어 플러그인이 없는
>        빈/오류 결과물로 덮어써 **사이트 전체 404 오류**를 유발합니다.
> 3. **에이전트 행동 지침**:
>    - 배포 오류나 404 문제를 디버깅할 때 `gh-pages` 브랜치 생성이나 `Deploy from a branch` 설정을
>      해결책으로 제안하거나 구성하지 마십시오.
>    - 배포 파이프라인의 수정은 오직 `.github/workflows/pages-deploy.yml`을 통해서만 수행합니다.

---

## 3. Directory Structure

```text
├── .github/workflows/
│   └── pages-deploy.yml     # GitHub Actions 배포 워크플로우
├── _config.yml              # Jekyll 및 Chirpy 사이트 전역 설정
├── _data/                   # 사이트 데이터 (share, contact, locales 등)
├── _includes/               # 재사용 HTML 컴포넌트
├── _plugins/                # Chirpy 테마 플러그인 (기본 GitHub Jekyll 빌더 호환 불가 원인)
├── _posts/                  # 블로그 포스트 마크다운 파일 (YYYY-MM-DD-title.md)
├── _tabs/                   # 상단/사이드바 탭 네비게이션 (about, archives, categories 등)
├── assets/                  # 정적 리소스 (img, CSS, JS)
├── Gemfile / Gemfile.lock   # Ruby 의존성 관리
└── index.html               # 메인 페이지 진입점
```

---

## 4. Content Creation & Post Guidelines

새로운 포스트를 작성하거나 기존 포스트를 수정할 때는 아래 규칙을 엄격히 준수합니다.

### 4.1 파일 명명 규칙

- 경로: `_posts/YYYY-MM-DD-title.md`
- 파일명은 반드시 소문자 영문, 숫자, 하이픈(`-`) 또는 언더스코어(`_`) 조합을 권장합니다.
- 예: `_posts/2025-09-05-Clean_architecture.md`

### 4.2 Front Matter 규격

모든 포스트는 상단에 올바른 YAML Front Matter를 포함해야 합니다:

```yaml
---
title: "포스트 제목"
date: 2026-09-23 14:00:00 +0900 # 타임존 명시 필수 (+0900 등). 미래 날짜 금지 (Jekyll이 빌드에서 제외함)
categories: [대분류, 소분류] # 최대 2단계 계층 권장
tags: [tag1, tag2, tag3] # 소문자 권장
math: true # 수식(MathJax) 필요 시 true
pin: false # 메인 상단 고정 여부
toc: true # 목차 표시 여부
---
```

### 4.3 링크 및 이미지 규칙 (HTMLProofer 통과 필수)

- **내부 링크**: 깨진 내부 앵커나 상대 경로 링크가 없어야 합니다. 워크플로우에서 `htmlproofer`
  검사가 실행되므로 깨진 링크가 하나라도 있으면 빌드가 실패합니다.
- **이미지 경로**: `/assets/img/...` 절대 경로 또는 유효한 상대 경로를 사용합니다.
- **외부 URL**: 워크플로우에서 `--disable-external`이 적용되어 있으나 유효하고 안전한 URL만
  사용합니다.

---

## 5. Development & Verification Commands

로컬 개발 환경(WSL, DevContainer, Ruby 환경 등)에서 작업할 때 다음 명령어를 사용합니다:

```bash
# 의존성 설치
bundle install

# 로컬 개발 서버 구동 (실시간 리로드)
bundle exec jekyll serve

# 로컬 환경에서 초안(drafts) 및 미래 포스트 포함하여 서빙
bundle exec jekyll serve --drafts --future

# 프로덕션 빌드 테스트 (CI 환경과 동일)
JEKYLL_ENV=production bundle exec jekyll build -d _site

# HTML & 링크 유효성 검사 (CI 검증과 동일)
bundle exec htmlproofer _site \
  --disable-external \
  --ignore-urls "/^http:\/\/127.0.0.1/,/^http:\/\/0.0.0.0/,/^http:\/\/localhost/"
```

---

## 6. Rules for Commits & Changes

1. **\_site 및 .jekyll-cache 주의**:
   - 빌드 산출물(`_site/`) 및 캐시 폴더는 git에 커밋되지 않도록 `.gitignore` 상태를 확인하십시오.
2. **배포 워크플로우 변경 시**:
   - `.github/workflows/pages-deploy.yml` 내 `concurrency`,
     `permissions(pages: write, id-token: write)` 및 빌드 스텝을 훼손하지 마십시오.
3. **인코딩**:
   - 모든 파일은 UTF-8 인코딩을 유지해야 합니다.
