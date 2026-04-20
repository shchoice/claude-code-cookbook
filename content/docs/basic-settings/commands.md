---
title: 명령어 (Commands)
weight: 2
sources:
  - title: "Claude Code 공식 문서 - Commands"
    url: "https://code.claude.com/docs/en/commands"
---

Claude Code 대화 중 `/`로 시작하는 슬래시 명령어와, 터미널에서 `claude` 실행 시 사용하는 CLI 플래그를 정리합니다. 세션 관리, 모델 전환, 권한 설정, 코드 리뷰, 자동화까지 대부분의 조작을 명령어로 할 수 있습니다.

대화 중 `/`를 입력하면 사용 가능한 명령어 목록이 나타납니다. 글자를 이어 입력하면 필터링됩니다.

> `<arg>`는 필수, `[arg]`는 선택 인자입니다. 플랜, 플랫폼, 환경에 따라 보이지 않는 명령어도 있습니다.

---

## 세션 관리

다른 작업으로 넘어갈 때, 어제 하던 걸 이어할 때, 대화가 길어져서 성능이 떨어질 때 씁니다.

| 명령어 | 설명 |
|--------|------|
| `/clear` | 대화 초기화. 다른 주제로 넘어갈 때 컨텍스트를 비움. 이전 세션은 `/resume`로 복구 가능. `/reset`, `/new` |
| `/compact [instructions]` | 대화를 요약해서 컨텍스트 확보. 대화가 길어졌을 때 씀. 예: `/compact 코드 변경에 집중해` |
| `/resume [session]` | 이전 세션 이어서 시작. 이름이나 ID로 지정하거나, 인자 없이 실행하면 세션 목록이 뜸. `/continue` |
| `/rename [name]` | 세션에 이름 붙이기. `/resume`에서 찾기 쉬워짐. 인자 없으면 대화 내용에서 자동 생성 |
| `/branch [name]` | 현재 대화에서 분기점 생성. 다른 접근 방식을 시도해보고 싶을 때. `/fork` |
| `/rewind` | 대화/코드를 이전 시점으로 되돌리기. Claude의 수정이 마음에 안 들 때. `Esc+Esc`으로도 실행 가능. `/checkpoint`, `/undo` |
| `/export [filename]` | 대화를 텍스트 파일로 내보내기. 대화 기록을 공유하거나 보관할 때 |
| `/copy [N]` | 마지막 응답 클립보드 복사. `N`으로 N번째 이전 응답 선택. 코드 블록이 있으면 개별 선택 가능 |
| `/btw <question>` | 대화에 안 남는 사이드 질문. 작업 흐름을 끊지 않고 잠깐 확인할 때. Claude가 작업 중일 때도 사용 가능 |

---

## 모델 및 모드

작업 난이도에 따라 모델이나 사고 깊이를 바꿀 때 씁니다.

| 명령어 | 설명 |
|--------|------|
| `/model [model]` | 모델 변경. 좌우 화살표로 effort도 조절 가능. 복잡한 설계는 opus, 일반 코딩은 sonnet |
| `/effort [level\|auto]` | 사고 깊이 설정 (`low`, `medium`, `high`, `xhigh`, `max`). 간단한 작업은 low, 복잡한 추론은 max |
| `/fast [on\|off]` | Fast 모드 토글. 같은 모델에서 응답 속도를 높임 |
| `/plan [description]` | plan 모드 진입. 큰 기능을 구현하기 전에 방향을 잡을 때. 예: `/plan 인증 버그 수정` |

---

## 권한 및 설정

Claude Code의 동작 방식을 바꿀 때 씁니다.

| 명령어 | 설명 |
|--------|------|
| `/permissions` | 어떤 도구가 허용/차단되어 있는지 확인하거나 규칙 추가. auto 모드에서 거부된 동작도 여기서 확인. `/allowed-tools` |
| `/config` | 테마, 사고 모드, 에디터 모드, 세션 요약 등 각종 설정을 한곳에서 변경. `/settings` |
| `/sandbox` | 샌드박스 모드 토글. Bash 명령어의 파일/네트워크 접근을 OS 수준으로 격리 |
| `/theme` | 색상 테마 변경. auto 선택 시 터미널의 다크/라이트 모드를 자동으로 따라감 |
| `/color [color]` | 프롬프트 바 색상 변경. 여러 세션을 구분할 때 유용 |
| `/keybindings` | 키바인딩 설정 파일 열기. 단축키를 커스터마이징할 때 |

---

## 프로젝트 설정

프로젝트에 맞게 Claude Code를 커스터마이징할 때 씁니다.

| 명령어 | 설명 |
|--------|------|
| `/init` | 새 프로젝트에서 CLAUDE.md 자동 생성. 빌드 시스템, 테스트 프레임워크를 분석해서 초안을 만들어 줌 |
| `/memory` | CLAUDE.md 편집 및 auto-memory 관리. Claude가 기억할 규칙을 추가/수정할 때 |
| `/add-dir <path>` | 작업 디렉토리 추가. 다른 프로젝트의 파일도 읽고 수정해야 할 때 |
| `/hooks` | 현재 설정된 Hook 목록 확인. 어떤 이벤트에 어떤 스크립트가 걸려 있는지 |
| `/skills` | 사용 가능한 skill 목록. `t`를 누르면 토큰 사용량 순으로 정렬 |
| `/agents` | 서브에이전트 생성/관리. 전문 분야별 에이전트를 만들 때 |
| `/plugin` | 플러그인 설치/관리. 코드 인텔리전스, 외부 도구 연동 등 |
| `/mcp` | MCP 서버 연결 관리. Notion, Figma, DB 등 외부 서비스와 연결할 때 |

---

## 코드 리뷰 및 작업

코드를 검토하거나, 대규모 변경을 수행할 때 씁니다.

| 명령어 | 설명 |
|--------|------|
| `/review [PR]` | PR 코드 리뷰 (로컬). 현재 브랜치의 PR을 자동 감지하거나, PR 번호/URL 지정 |
| `/ultrareview [PR]` | 클라우드에서 멀티 에이전트가 심층 리뷰. 로컬 `/review`보다 깊은 분석이 필요할 때 |
| `/ultraplan <prompt>` | 브라우저에서 플랜을 리뷰한 뒤 실행. 복잡한 계획을 시각적으로 검토할 때 |
| `/diff` | Claude가 뭘 수정했는지 확인할 때. 좌우 화살표로 git diff ↔ 턴별 diff 전환, 상하로 파일 탐색 |
| `/simplify [focus]` | 최근 변경 파일의 코드 품질을 자동 개선. 3개 리뷰 에이전트가 병렬로 분석. Skill |
| `/batch <instruction>` | 대규모 병렬 코드 변경. 5~30개 단위로 분할해서 각각 worktree에서 작업 후 PR 생성. Skill |
| `/security-review` | 현재 브랜치의 보안 취약점 분석. 인젝션, 인증 문제, 데이터 노출 등을 검사 |

---

## 정보 확인

현재 상태를 파악하거나 문제를 진단할 때 씁니다.

| 명령어 | 설명 |
|--------|------|
| `/status` | 현재 모델, 버전, 계정, 연결 상태 확인. Claude가 응답 중에도 실행 가능 |
| `/cost` | 이번 세션의 토큰 사용량과 예상 비용 확인. API 사용자용 (Pro/Max 구독자는 `/stats`) |
| `/usage` | 플랜 사용량과 속도 제한 상태 확인. 속도 제한에 걸렸을 때 남은 한도를 확인할 때 |
| `/stats` | 일별 사용량, 세션 히스토리, 모델 선호도 시각화. 내 사용 패턴을 파악할 때 |
| `/context` | 컨텍스트가 얼마나 찼는지 색상 격자로 시각화. 무엇이 컨텍스트를 많이 차지하는지, 최적화 제안도 표시 |
| `/doctor` | 설치 상태와 설정을 진단. 뭔가 이상할 때 먼저 실행. `f`를 누르면 발견된 문제를 자동 수정 |
| `/recap` | 현재 세션의 한 줄 요약. 자리를 비웠다 돌아왔을 때 상황 파악용 |

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
| `/help` | 단축키와 명령어를 한눈에 확인. 뭘 할 수 있는지 모를 때 가장 먼저 실행 |
| `/feedback [report]` | Claude Code 버그나 피드백 제출. `/bug` |
| `/release-notes` | 버전별 변경사항 확인. 업데이트 후 뭐가 바뀌었는지 볼 때 |
| `/powerup` | 기능 소개 인터랙티브 데모. Claude Code 입문자에게 추천 |
| `/debug [description]` | 디버그 로그 활성화 및 분석. 문제 상황을 재현하고 원인을 파악할 때. Skill |
| `/tui [default\|fullscreen]` | 터미널 UI 렌더러 변경. fullscreen은 화면 깜빡임을 줄이고 마우스 지원 추가 |
| `/statusline` | 상태 표시줄에 모델, 컨텍스트 사용량 등을 표시. 인자 없이 실행하면 쉘 프롬프트 기반으로 자동 설정 |
| `/terminal-setup` | Shift+Enter 등 터미널 키바인딩 자동 설정. VS Code, Alacritty, Zed, Warp 사용자용 |
| `/fewer-permission-prompts` | "Allow" 클릭이 귀찮을 때. 자주 쓰는 읽기 전용 명령어를 자동으로 allowlist에 추가. Skill |
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

---

## 처음 써본다면

1. `/help`로 어떤 명령어가 있는지 훑어보기
2. `/model`로 모델 확인하고, `/effort`로 사고 깊이 확인
3. 작업 중간에 `/context`로 컨텍스트 사용량 체크
4. 다른 작업으로 넘어갈 때 `/clear`로 초기화
5. 뭔가 이상하면 `/doctor`로 진단
