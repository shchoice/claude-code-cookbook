---
title: 명령어 (Commands)
weight: 2
sources:
  - title: "Claude Code 공식 문서 - Commands"
    url: "https://code.claude.com/docs/en/commands"
---

대화 중 `/`를 입력하면 사용 가능한 명령어 목록이 나타납니다. 글자를 이어 입력하면 필터링됩니다.

> `<arg>`는 필수, `[arg]`는 선택 인자입니다. 플랜, 플랫폼, 환경에 따라 보이지 않는 명령어도 있습니다.

---

## 세션 관리

| 명령어 | 설명 |
|--------|------|
| `/clear` | 대화 초기화 (이전 세션은 `/resume`로 복구 가능). `/reset`, `/new` |
| `/compact [instructions]` | 대화 요약으로 컨텍스트 확보. 예: `/compact 코드 변경에 집중해` |
| `/resume [session]` | 이전 세션 이어서 시작. `/continue` |
| `/rename [name]` | 현재 세션 이름 지정 |
| `/branch [name]` | 대화 분기점 생성. `/fork` |
| `/rewind` | 대화/코드를 이전 시점으로 되돌리기. `/checkpoint`, `/undo` |
| `/export [filename]` | 대화를 텍스트 파일로 내보내기 |
| `/copy [N]` | 마지막 응답 클립보드 복사. `N`으로 N번째 이전 응답 선택 |
| `/btw <question>` | 대화에 안 남는 사이드 질문 |

---

## 모델 및 모드

| 명령어 | 설명 |
|--------|------|
| `/model [model]` | 모델 변경. 좌우 화살표로 effort 조절 |
| `/effort [level\|auto]` | 사고 깊이 설정 (`low`, `medium`, `high`, `xhigh`, `max`) |
| `/fast [on\|off]` | Fast 모드 토글 |
| `/plan [description]` | plan 모드 진입. 예: `/plan 인증 버그 수정` |

---

## 권한 및 설정

| 명령어 | 설명 |
|--------|------|
| `/permissions` | allow/ask/deny 규칙 관리. `/allowed-tools` |
| `/config` | 테마, 모델, 에디터 모드 등 설정. `/settings` |
| `/sandbox` | 샌드박스 모드 토글 |
| `/theme` | 색상 테마 변경 (auto, light, dark, 색각이상 대응 등) |
| `/color [color]` | 프롬프트 바 색상 변경 |
| `/keybindings` | 키바인딩 설정 파일 열기 |

---

## 프로젝트 설정

| 명령어 | 설명 |
|--------|------|
| `/init` | CLAUDE.md 자동 생성 |
| `/memory` | CLAUDE.md 편집, auto-memory 관리 |
| `/add-dir <path>` | 작업 디렉토리 추가 |
| `/hooks` | Hook 설정 확인 |
| `/skills` | 사용 가능한 skill 목록. `t`로 토큰 순 정렬 |
| `/agents` | 서브에이전트 관리 |
| `/plugin` | 플러그인 관리 |
| `/mcp` | MCP 서버 연결 관리 |

---

## 코드 리뷰 및 작업

| 명령어 | 설명 |
|--------|------|
| `/review [PR]` | PR 코드 리뷰 (로컬) |
| `/ultrareview [PR]` | 클라우드 기반 멀티 에이전트 심층 리뷰 |
| `/ultraplan <prompt>` | 브라우저에서 플랜 리뷰 후 실행 |
| `/diff` | 미커밋 변경사항 + 턴별 diff 뷰어 |
| `/simplify [focus]` | 최근 변경 파일의 코드 품질 개선. Skill |
| `/batch <instruction>` | 대규모 병렬 코드 변경. Skill |
| `/security-review` | 현재 브랜치의 보안 취약점 분석 |

---

## 정보 확인

| 명령어 | 설명 |
|--------|------|
| `/cost` | 토큰 사용량 (API 사용자용) |
| `/usage` | 플랜 사용량 및 속도 제한 상태 |
| `/stats` | 일별 사용량, 세션 히스토리, 모델 선호도 |
| `/status` | 버전, 모델, 계정, 연결 상태 |
| `/context` | 컨텍스트 사용량 시각화 |
| `/doctor` | 설치 및 설정 진단. `f`로 자동 수정 |
| `/recap` | 현재 세션 한 줄 요약 |

---

## 연동 및 계정

| 명령어 | 설명 |
|--------|------|
| `/login` | Anthropic 계정 로그인 |
| `/logout` | 로그아웃 |
| `/install-github-app` | GitHub Actions 앱 설정 |
| `/install-slack-app` | Slack 앱 설치 |
| `/web-setup` | Claude Code on the web용 GitHub 연결 |
| `/remote-control` | 현재 세션을 claude.ai에서 원격 제어. `/rc` |
| `/desktop` | Desktop 앱에서 현재 세션 이어서. `/app` |
| `/teleport` | 웹 세션을 터미널로 가져오기. `/tp` |
| `/ide` | IDE 연동 관리 |

---

## 자동화 및 스케줄

| 명령어 | 설명 |
|--------|------|
| `/loop [interval] [prompt]` | 프롬프트 반복 실행. 예: `/loop 5m 배포 확인해`. Skill |
| `/schedule [description]` | Routine 생성/관리. `/routines` |
| `/autofix-pr [prompt]` | PR의 CI 실패/리뷰 코멘트 자동 수정 |
| `/tasks` | 백그라운드 작업 관리 |

---

## 기타

| 명령어 | 설명 |
|--------|------|
| `/help` | 도움말 및 명령어 목록 |
| `/feedback [report]` | 피드백/버그 제출. `/bug` |
| `/release-notes` | 버전별 변경사항 확인 |
| `/powerup` | 기능 소개 인터랙티브 데모 |
| `/debug [description]` | 디버그 로그 활성화 및 분석. Skill |
| `/tui [default\|fullscreen]` | 터미널 UI 렌더러 변경 |
| `/statusline` | 상태 표시줄 설정 |
| `/terminal-setup` | Shift+Enter 등 터미널 키바인딩 설정 |
| `/fewer-permission-prompts` | 권한 프롬프트를 줄이는 allowlist 자동 생성. Skill |
| `/exit` | 종료. `/quit` |

---

## CLI 플래그

터미널에서 `claude` 실행 시 사용하는 옵션입니다. 자주 쓰는 것만 정리합니다.

```bash
# 비대화형 모드 — 결과만 출력하고 종료
claude -p "README.md를 한국어로 번역해줘"

# 모델 지정
claude --model opus

# 권한 모드 지정
claude --permission-mode plan

# 이전 대화 이어서
claude --continue

# 세션 선택해서 이어서
claude --resume

# 세션 이름 지정
claude -n auth-refactor

# Git worktree에서 격리 실행
claude --worktree feature-auth

# 출력 형식 지정 (스크립트 연동용)
claude -p "API 엔드포인트 목록" --output-format json
```

> 전체 CLI 플래그 목록은 `claude --help`로 확인할 수 있습니다.
