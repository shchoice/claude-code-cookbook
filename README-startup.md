# 시작 가이드 (README-startup.md)

처음 기여하는 분을 위한 카테고리 관리법, 글 작성법, 로컬 미리보기 안내입니다.

---

## 목차

1. [카테고리 관리법](#1-카테고리관리법)
2. [글 작성법](#2-글-작성법)
3. [로컬 미리보기](#3-로컬-미리보기)

---

## 1. 카테고리 관리법

카테고리는 `content/docs/` 아래의 폴더 단위로 관리합니다.

```
content/docs/
├── claude-code/       # 카테고리
│   ├── _index.md      # 카테고리 소개 페이지 (필수)
│   └── ...
└── ai-agent-rag/      # 카테고리
    ├── _index.md
    └── ...
```

### 새 카테고리 만들기

폴더를 만들고 반드시 `_index.md`를 함께 생성합니다.

```
content/docs/llm-prompting/_index.md
```

```markdown
---
title: LLM 프롬프팅
weight: 3
---

카테고리 설명을 여기에 씁니다.
```

| 필드 | 설명 |
|------|------|
| `title` | 사이드바에 표시될 카테고리 이름 |
| `weight` | 사이드바 정렬 순서. 숫자가 낮을수록 위에 표시됨 |

> `_index.md`가 없으면 폴더 안에 파일이 있어도 사이드바에 나타나지 않습니다.

### 카테고리 순서 변경

각 카테고리의 `_index.md`에서 `weight` 값을 조정합니다.

| 카테고리 | weight |
|--------|--------|
| Claude Code | 1 |
| AI Agent & RAG | 2 |

---

## 2. 글 작성법

### 블로그 게시물

**파일명 규칙**: `content/blog/YYYY-MM-DD-슬러그.md`

```
content/blog/2026-04-16-claude-code-hooks-guide.md
```

```markdown
---
title: "Claude Code Hooks 완벽 가이드"
date: 2026-04-16
draft: false
tags: ["claude-code", "hooks"]
description: "훅을 이용해 워크플로우를 자동화하는 방법을 설명합니다."
---

본문을 여기에 작성합니다.
```

| 필드 | 필수 | 설명 |
|------|:----:|------|
| `title` | ✅ | 게시물 제목 |
| `date` | ✅ | 작성 날짜 (`YYYY-MM-DD`) |
| `draft` | ✅ | `true`면 배포 제외, `false`면 배포 포함 |
| `tags` | - | 태그 목록 |
| `description` | - | 목록 페이지·SEO용 요약문 |

---

### 문서(docs) 게시물

**파일명 규칙**: `content/docs/카테고리명/파일명.md`

```
content/docs/claude-code/hooks.md
```

```markdown
---
title: "Hooks 설정하기"
weight: 4
---

본문을 여기에 작성합니다.
```

| 필드 | 필수 | 설명 |
|------|:----:|------|
| `title` | ✅ | 페이지 제목 |
| `weight` | - | 사이드바 내 정렬 순서 (낮을수록 위) |
| `draft` | - | `true`면 배포 제외 |

---

### 이미지 삽입

이미지는 `static/images/`에 저장한 뒤 아래와 같이 참조합니다.

```markdown
![설명](/images/파일명.png)
```

---

## 3. 로컬 미리보기

```bash
# 기본 실행
hugo server

# draft: true인 글도 함께 보기
hugo server -D

# 미래 날짜 글도 함께 보기
hugo server -D -F
```

브라우저에서 아래 주소로 접속합니다.

```
http://localhost:1313/claude-code-cookbook
```

파일을 저장하면 브라우저가 자동으로 새로고침됩니다. 종료는 `Ctrl + C`입니다.
