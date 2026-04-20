---
title: 자주 쓰는 단축키
weight: 6
sources:
  - title: "Claude Code 공식 문서 - Interactive Mode"
    url: "https://code.claude.com/docs/en/interactive-mode"
  - title: "Claude Code 공식 문서 - Keybindings"
    url: "https://code.claude.com/docs/en/keybindings"
  - title: "Claude Code 공식 문서 - Terminal Config"
    url: "https://code.claude.com/docs/en/terminal-config"
---

Claude Code 대화 중에 쓸 수 있는 키보드 단축키와 자주 쓰는 슬래시 명령어를 정리합니다. 작업 중단(`Ctrl+C`), 모드 전환(`Shift+Tab`), 사고 모드 토글(`Option+T`), 줄바꿈(`Ctrl+J`) 등 핵심 조작부터, 상황별로 어떤 명령어를 써야 하는지까지 한눈에 볼 수 있습니다.

`?`를 누르면 현재 환경에서 사용 가능한 단축키를 확인할 수 있습니다.

> macOS에서 `Option`/`Alt` 키 단축키를 사용하려면 터미널에서 Option을 Meta 키로 설정해야 합니다.

| 터미널 | 설정 경로 |
|--------|-----------|
| Terminal.app | Settings → Profiles → Keyboard → "Use Option as Meta Key" |
| iTerm2 | Settings → Profiles → Keys → Left/Right Option key → "Esc+" |
| VS Code | `terminal.integrated.macOptionIsMeta: true` |

---

## 일반 제어

| 단축키 | 동작 |
|--------|------|
| `Enter` | 메시지 전송 |
| `Ctrl + C` | 현재 작업 중단 |
| `Ctrl + D` | Claude Code 종료 |
| `Escape` | 입력 취소 |
| `Esc + Esc` | 되감기(rewind) — 대화/코드를 이전 시점으로 복원 |
| `Ctrl + L` | 입력창 비우기 + 화면 다시 그리기 |
| `Ctrl + O` | transcript 뷰어 토글 (상세 도구 사용 로그) |
| `Ctrl + R` | 히스토리 역방향 검색 |
| `↑` / `↓` | 이전/다음 히스토리 (멀티라인에서는 커서 이동 우선) |

---

## 모드 및 모델

| 단축키 | 동작 |
|--------|------|
| `Shift + Tab` | 권한 모드 순환 (default → acceptEdits → plan → ...) |
| `Option + P` / `Alt + P` | 모델 전환 |
| `Option + T` / `Alt + T` | 사고 모드 (Extended Thinking) 토글 |
| `Option + O` / `Alt + O` | Fast 모드 토글 |

사고 깊이 조절, UltraThink 등 심화 내용은 [Claude Code 더 잘 사용하기]({{< relref "/docs/tips/better-usage#사고-깊이-조절하기" >}})를 참고하세요.

---

## 텍스트 편집

| 단축키 | 동작 |
|--------|------|
| `Ctrl + A` | 줄 시작으로 이동 |
| `Ctrl + E` | 줄 끝으로 이동 |
| `Ctrl + K` | 커서부터 줄 끝까지 삭제 |
| `Ctrl + U` | 커서부터 줄 시작까지 삭제 |
| `Ctrl + W` | 이전 단어 삭제 |
| `Ctrl + Y` | 삭제한 텍스트 붙여넣기 |
| `Alt + Y` | 붙여넣기 히스토리 순환 (`Ctrl+Y` 직후) |
| `Alt + B` | 한 단어 뒤로 이동 |
| `Alt + F` | 한 단어 앞으로 이동 |

> `Ctrl+K/U/W`로 삭제한 텍스트는 `Ctrl+Y`로 복원할 수 있습니다 (kill ring).

---

## 줄바꿈 입력

메시지 안에서 줄을 바꾸는 방법입니다.

| 방법 | 단축키 | 비고 |
|------|--------|------|
| 모든 터미널 | `\` + `Enter` | 가장 확실한 방법 |
| 모든 터미널 | `Ctrl + J` | line feed 문자 |
| macOS | `Option + Enter` | Option as Meta 설정 필요 |
| iTerm2, WezTerm, Ghostty, Kitty | `Shift + Enter` | 설정 없이 바로 동작 |
| VS Code, Alacritty, Zed, Warp | `Shift + Enter` | `/terminal-setup` 실행 후 사용 가능 |

---

## 빠른 명령어

| 접두사 | 동작 | 예시 |
|--------|------|------|
| `/` | 슬래시 명령어/스킬 실행 | `/compact`, `/model opus` |
| `!` | Bash 명령어 직접 실행 | `! npm test`, `! git status` |
| `@` | 파일 경로 자동완성 | `@src/auth.js 설명해줘` |

`!`로 실행한 명령어의 출력은 대화 컨텍스트에 포함됩니다. Claude의 해석이나 승인 없이 바로 실행됩니다.

### 상황별 자주 쓰는 명령어

> 전체 명령어 목록은 [명령어 (Commands)]({{< relref "commands" >}}) 페이지를 참고하세요.

**처음 시작할 때**

| 명령어 | 상황 |
|--------|------|
| `/help` | 어떤 명령어가 있는지 모를 때. 단축키와 명령어 목록을 한눈에 볼 수 있음 |
| `/init` | 새 프로젝트를 시작할 때. CLAUDE.md를 자동 생성해서 Claude가 프로젝트를 이해하게 함 |
| `/doctor` | Claude Code가 이상하게 동작할 때. 설치 상태와 설정을 진단하고, `f`를 눌러 자동 수정 |
| `/terminal-setup` | Shift+Enter가 안 먹힐 때. VS Code, Alacritty 등에서 키바인딩을 자동 설정 |

**작업 중에 자주 쓰는 것**

| 명령어 | 상황 |
|--------|------|
| `/clear` | 다른 주제로 넘어갈 때. 컨텍스트를 비워서 성능 유지 |
| `/compact` | 대화가 길어졌을 때. 요약해서 컨텍스트를 확보. 예: `/compact 테스트 결과에 집중해` |
| `/model` | 모델을 바꾸고 싶을 때. 좌우 화살표로 effort도 조절 가능 |
| `/config` | 테마, 사고 모드, 에디터 모드 등 각종 설정을 변경할 때 |
| `/diff` | Claude가 뭘 수정했는지 확인할 때. 턴별 diff도 볼 수 있음 |
| `/rewind` | Claude의 수정이 마음에 안 들 때. 대화와 코드를 이전 시점으로 되돌림 |

**세션 관리**

| 명령어 | 상황 |
|--------|------|
| `/resume` | 어제 하던 작업을 이어서 할 때. 세션 목록에서 선택 가능 |
| `/rename` | 세션에 이름을 붙여둘 때. 나중에 `/resume`에서 쉽게 찾을 수 있음 |
| `/btw` | 작업 흐름을 끊지 않고 잠깐 질문할 때. 대화 기록에 남지 않음 |

**상태 확인**

| 명령어 | 상황 |
|--------|------|
| `/status` | 현재 모델, 버전, 계정 정보를 확인할 때 |
| `/cost` | 이번 세션에서 토큰을 얼마나 썼는지 확인 (API 사용자용) |
| `/usage` | 플랜 사용량과 속도 제한 상태를 확인할 때 |
| `/context` | 컨텍스트가 얼마나 찼는지 시각적으로 확인. 최적화 제안도 표시 |

**권한 관련**

| 명령어 | 상황 |
|--------|------|
| `/permissions` | 어떤 도구가 허용/차단되어 있는지 확인하거나 규칙을 추가할 때 |
| `/fewer-permission-prompts` | "Allow" 버튼 누르는 게 귀찮을 때. 자주 쓰는 명령어를 자동으로 allowlist에 추가 |

---

## 이미지 및 파일

| 단축키 | 동작 |
|--------|------|
| `Ctrl + V` | 클립보드 이미지 붙여넣기 (Windows: `Alt + V`) |
| `Ctrl + G` | 외부 에디터로 열기 (plan 파일 편집 시 유용) |
| `Ctrl + S` | 현재 프롬프트 임시 저장 (stash) |

---

## 백그라운드 작업

| 단축키 | 동작 |
|--------|------|
| `Ctrl + B` | 실행 중인 작업을 백그라운드로 보내기 (tmux에서는 2번) |
| `Ctrl + X → Ctrl + K` | 모든 백그라운드 에이전트 종료 (3초 내 2번 눌러 확인) |
| `Ctrl + T` | 작업 목록(Task) 토글 |

---

## 권한 확인 대화상자

권한 확인 팝업이 뜨면 사용할 수 있는 단축키입니다.

| 단축키 | 동작 |
|--------|------|
| `Y` 또는 `Enter` | 허용 |
| `N` 또는 `Escape` | 거부 |
| `Shift + Tab` | 권한 모드 전환 |
| `Ctrl + E` | 권한 설명 토글 |
| `←` / `→` | 탭 간 이동 |

---

## Transcript 뷰어

`Ctrl + O`로 transcript 뷰어를 연 상태에서 사용할 수 있습니다.

| 단축키 | 동작 |
|--------|------|
| `Ctrl + E` | 전체 내용 표시 토글 |
| `q` / `Ctrl + C` / `Esc` | 뷰어 닫기 |

---

## 음성 입력

음성 입력(`/voice`로 활성화)이 켜진 상태에서:

| 단축키 | 동작 |
|--------|------|
| `Space` 길게 누르기 | 푸시투톡 — 누르는 동안 음성 인식 |

---

## 단축키 커스터마이징

기본 단축키를 바꾸고 싶다면 `/keybindings`를 실행합니다. `~/.claude/keybindings.json` 파일이 열립니다.

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null
      }
    }
  ]
}
```

| 문법 | 예시 | 의미 |
|------|------|------|
| 수정자 + 키 | `ctrl+k` | Ctrl과 K 동시 |
| 코드(chord) | `ctrl+k ctrl+s` | Ctrl+K 누른 뒤 Ctrl+S |
| `null` | `"ctrl+u": null` | 해당 단축키 해제 |

> 변경사항은 파일 저장 즉시 적용됩니다. 재시작 불필요.

### 변경 불가능한 단축키

| 단축키 | 이유 |
|--------|------|
| `Ctrl + C` | 시스템 인터럽트 |
| `Ctrl + D` | 시스템 종료 |
| `Ctrl + M` | 터미널에서 Enter와 동일 |
