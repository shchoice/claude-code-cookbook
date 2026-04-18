---
title: 자주 쓰는 단축키
weight: 6
sources:
  - title: "Claude Code 공식 문서 - Keybindings"
    url: "https://code.claude.com/docs/en/keybindings"
  - title: "Claude Code 공식 문서 - Terminal Config"
    url: "https://code.claude.com/docs/en/terminal-config"
  - title: "Claude Code 공식 문서 - Common Workflows"
    url: "https://code.claude.com/docs/en/common-workflows"
---

Claude Code 대화 중에 쓸 수 있는 단축키를 정리했습니다. 마우스 없이 키보드만으로 대부분의 조작이 가능합니다.

## 필수 단축키

매일 쓰게 되는 핵심 단축키입니다.

| 단축키 | 동작 | 설명 |
|--------|------|------|
| `Enter` | 메시지 전송 | |
| `Ctrl + C` | 현재 작업 중단 | Claude가 작업 중일 때 멈춤 |
| `Ctrl + D` | Claude Code 종료 | |
| `Shift + Tab` | 권한 모드 전환 | default → acceptEdits → plan → ... |
| `Ctrl + J` | 줄바꿈 | 메시지 안에서 엔터 없이 다음 줄로 |
| `Escape` | 입력 취소 | 작성 중인 메시지 지우기 |

---

## 사고 모드 (Thinking) 제어

| 단축키 | 동작 |
|--------|------|
| `Option + T` (macOS) / `Alt + T` | 사고 모드 켜기/끄기 (현재 세션) |
| `Ctrl + O` | verbose 모드 — 사고 과정 보기 |

> macOS에서 `Option` 키가 동작하지 않으면 터미널 설정에서 "Use Option as Meta Key"를 켜야 합니다.

| 터미널 | 설정 경로 |
|--------|-----------|
| Terminal.app | Settings → Profiles → Keyboard → "Use Option as Meta Key" |
| iTerm2 | Settings → Profiles → Keys → Left/Right Option key → "Esc+" |
| VS Code | `terminal.integrated.macOptionIsMeta: true` |

단축키 없이 영구 설정하려면: `/config` → Think Mode → false

사고 깊이 조절, UltraThink, effort level 등 심화 내용은 [Claude Code 더 잘 사용하기]({{< relref "/docs/tips/better-usage#사고-깊이-조절하기" >}})를 참고하세요.

---

## 모드 및 모델

| 단축키 | 동작 |
|--------|------|
| `Shift + Tab` | 권한 모드 순환 (default → acceptEdits → plan → ...) |
| `Cmd + P` / `Meta + P` | 모델 선택 |
| `Meta + O` | Fast 모드 토글 |

---

## 편집 및 입력

| 단축키 | 동작 |
|--------|------|
| `Ctrl + J` | 줄바꿈 삽입 |
| `Ctrl + L` | 입력창 비우기 + 화면 다시 그리기 |
| `Ctrl + G` | 외부 에디터로 열기 (plan 파일 편집 시 유용) |
| `Ctrl + V` | 이미지 붙여넣기 (Windows: `Alt + V`) |
| `Ctrl + S` | 현재 프롬프트 임시 저장 (stash) |
| `Ctrl + _` | 실행 취소 (undo) |

---

## 세션 관리

| 단축키 | 동작 |
|--------|------|
| `Ctrl + R` | 히스토리 검색 |
| `↑` / `↓` | 이전/다음 히스토리 |
| `Ctrl + O` | verbose 모드 토글 (상세 로그 표시) |
| `Ctrl + T` | 작업 목록(Task) 토글 |

---

## 백그라운드 작업

| 단축키 | 동작 |
|--------|------|
| `Ctrl + B` | 현재 작업을 백그라운드로 보내기 |
| `Ctrl + X → Ctrl + K` | 모든 백그라운드 에이전트 종료 |

---

## 권한 확인 대화상자

권한 확인 팝업이 뜨면 사용할 수 있는 단축키입니다.

| 단축키 | 동작 |
|--------|------|
| `Y` 또는 `Enter` | 허용 |
| `N` 또는 `Escape` | 거부 |
| `Shift + Tab` | 권한 모드 전환 |
| `Ctrl + E` | 권한 설명 토글 |

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
