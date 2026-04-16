---
title: "Claude Code로 블로그 자동화하기"
date: 2026-04-13
draft: false
tags: ["claude-code", "automation", "hugo"]
description: "Claude Code를 이용해 Hugo 블로그 게시물 작성과 배포를 자동화한 경험을 공유합니다."
---

## 배경

Hugo로 기술 블로그를 운영하다 보면 반복되는 작업이 많습니다. 게시물 파일 생성, front matter 작성, 로컬 확인, 빌드, 배포... Claude Code를 활용하면 이 과정을 대폭 줄일 수 있습니다.

## 핵심 아이디어

Claude Code에게 "블로그 게시물을 작성해줘"라고 하면, 단순히 텍스트만 쓰는 게 아니라 **파일 생성 → front matter 설정 → 내용 작성**까지 한 번에 처리합니다.

```bash
# 프로젝트 루트에서 Claude Code 실행
claude

# 그 다음 대화로
> content/blog/ 아래에 "Claude Code 시작하기" 제목으로 오늘 날짜 게시물 파일을 만들어줘
```

## 실전 워크플로우

1. **기획**: 주제만 던져주면 구조를 잡아줌
2. **작성**: 초안 생성 후 피드백으로 수정
3. **확인**: `hugo server` 로컬 미리보기
4. **배포**: git push 하면 GitHub Actions로 자동 배포

## 마무리

반복 작업은 Claude Code에게, 콘텐츠의 방향과 검수는 내가 — 이 조합이 생각보다 훨씬 잘 맞습니다.
