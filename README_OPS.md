# 운영 가이드 (README_OPS.md)

Hugo + Hextra 테마 기반 블로그의 게시물 작성 및 배포 방법을 설명합니다.

---

## 목차

1. [콘텐츠 구조](#1-콘텐츠-구조)
2. [게시물 종류별 작성법](#2-게시물-종류별-작성법)
3. [Front Matter 레퍼런스](#3-front-matter-레퍼런스)
4. [로컬에서 확인하기](#4-로컬에서-확인하기)
5. [배포하기](#5-배포하기)
6. [자주 하는 실수](#6-자주-하는-실수)

---

## 1. 콘텐츠 구조

```
content/
├── _index.md                        # 홈페이지
├── blog/
│   ├── _index.md                    # 블로그 목록 페이지
│   └── YYYY-MM-DD-slug.md           # 블로그 게시물
└── docs/
    ├── _index.md                    # 문서 목록 페이지
    ├── claude-code/                 # Claude Code 가이드 섹션
    │   ├── _index.md
    │   ├── installation.md
    │   └── commands.md
    └── ai-agent-rag/                # AI Agent & RAG 섹션
        ├── _index.md
        ├── concepts.md
        └── rag-patterns.md
```

**블로그** (`/blog`): 날짜 기반의 글. 경험 공유, 튜토리얼, 팁 등

**docs/claude-code** (`/docs/claude-code`): Claude Code 설치, 명령어, 활용 패턴 등 레퍼런스 문서

**docs/ai-agent-rag** (`/docs/ai-agent-rag`): AI Agent 설계 패턴, RAG 구현 레시피 등

---

## 2. 게시물 종류별 작성법

### 블로그 게시물 작성

**파일명 규칙**: `content/blog/YYYY-MM-DD-제목-슬러그.md`

```bash
# 예시
content/blog/2026-04-13-claude-code-tips.md
```

**파일 내용 예시**:

```markdown
---
title: "Claude Code 실전 팁 5가지"
date: 2026-04-13
draft: false
tags: ["claude-code", "tips"]
description: "매일 쓰면서 발견한 유용한 팁들을 정리했습니다."
---

본문 내용을 여기에 작성합니다.
```

---

### Claude Code 문서 작성

**파일명 규칙**: `content/docs/claude-code/파일명.md`

```bash
# 예시
content/docs/claude-code/claude-md.md
```

**파일 내용 예시**:

```markdown
---
title: CLAUDE.md 작성법
weight: 3
---

본문 내용을 여기에 작성합니다.
```

---

### AI Agent & RAG 문서 작성

**파일명 규칙**: `content/docs/ai-agent-rag/파일명.md`

```bash
# 예시
content/docs/ai-agent-rag/vector-db.md
```

**파일 내용 예시**:

```markdown
---
title: 벡터 DB 선택 가이드
weight: 3
---

본문 내용을 여기에 작성합니다.
```

---

### 새 섹션(폴더) 추가

`docs` 아래에 새 섹션을 추가할 때는 반드시 `_index.md`를 함께 만들어야 합니다.

```
content/docs/새섹션/_index.md
```

```markdown
---
title: 새 섹션 제목
weight: 3
---

섹션 설명 (선택사항)
```

---

## 3. Front Matter 레퍼런스

### 블로그 게시물 Front Matter

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `title` | ✅ | 게시물 제목 | `"Claude Code 팁"` |
| `date` | ✅ | 작성 날짜 | `2026-04-13` |
| `draft` | ✅ | `true`이면 배포 시 제외됨 | `false` |
| `tags` | - | 태그 목록 | `["claude-code", "tips"]` |
| `description` | - | 목록/SEO용 요약 | `"요약 설명"` |

### 문서 페이지 Front Matter

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `title` | ✅ | 페이지 제목 | `"설치 방법"` |
| `weight` | - | 사이드바 정렬 순서 (낮을수록 위) | `1` |
| `draft` | - | `true`이면 배포 시 제외됨 | `false` |

---

## 4. 로컬에서 확인하기

### Hugo 설치 (최초 1회)

```bash
# macOS
brew install hugo

# 설치 확인
hugo version
```

### 로컬 서버 실행

```bash
# 프로젝트 루트에서 실행
hugo server

# draft 상태인 게시물도 보고 싶을 때
hugo server -D
```

브라우저에서 http://localhost:1313/claude-code-cookbook 접속하여 확인합니다.

파일을 저장하면 **자동으로 새로고침**됩니다.

### 서버 종료

```
Ctrl + C
```

---

## 5. 배포하기

```bash
# 변경된 파일을 git에 추가
git add content/

# 커밋
git commit -m "docs: 새 게시물 추가 - 제목"

# 푸시 (GitHub Actions가 자동으로 빌드 & 배포)
git push
```

배포 후 https://shchoice.github.io/claude-code-cookbook 에서 확인합니다.

> **주의**: `draft: true` 로 설정된 게시물은 배포되지 않습니다. 배포하려면 `draft: false` 로 변경하세요.

---

## 6. 자주 하는 실수

### 게시물이 보이지 않을 때

- `draft: true` → `draft: false` 로 변경했는지 확인
- `date`가 미래 날짜로 설정되어 있으면 로컬에서도 안 보임 (`hugo server -F` 로 확인 가능)
- 새 폴더를 만들었는데 `_index.md`가 없으면 섹션이 나타나지 않음

### 빌드 에러가 날 때

```bash
# 의존성 재설치
hugo mod tidy

# 캐시 초기화 후 재시도
hugo mod clean
hugo server
```

### 이미지 추가 방법

이미지는 `static/images/` 폴더에 저장하고 마크다운에서 아래와 같이 참조합니다.

```markdown
![설명](/images/파일명.png)
```
