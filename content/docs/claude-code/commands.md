---
title: 자주 쓰는 명령어
weight: 2
---

## 슬래시 커맨드

Claude Code 대화 중 `/` 로 시작하는 내장 명령어입니다.

| 명령어 | 설명 |
|--------|------|
| `/help` | 사용 가능한 명령어 목록 |
| `/clear` | 대화 기록 초기화 |
| `/compact` | 대화 내용을 요약하여 컨텍스트 절약 |
| `/cost` | 현재 세션의 토큰 사용량 확인 |
| `/review` | 현재 변경사항 코드 리뷰 요청 |

## CLI 플래그

터미널에서 `claude` 실행 시 사용할 수 있는 옵션입니다.

```bash
# 특정 작업을 대화 없이 바로 실행 (비대화형 모드)
claude -p "README.md 파일을 한국어로 번역해줘"

# 특정 모델 지정
claude --model claude-opus-4-6

# 대화 이어서 시작
claude --continue
```

## CLAUDE.md 활용

프로젝트 루트에 `CLAUDE.md` 파일을 만들면 Claude Code가 자동으로 읽어 컨텍스트로 활용합니다.

```markdown
# 프로젝트 개요
이 프로젝트는 ...

# 코딩 컨벤션
- 들여쓰기: 2 스페이스
- 변수명: camelCase

# 자주 쓰는 명령어
- 빌드: npm run build
- 테스트: npm test
```
