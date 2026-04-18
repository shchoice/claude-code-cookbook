---
title: 설정 파일 (Settings)
weight: 3
sources:
  - title: "Claude Code 공식 문서 - Settings"
    url: "https://code.claude.com/docs/en/settings"
---

Claude Code의 동작은 `settings.json` 파일로 제어합니다. 스마트폰의 "설정 앱"과 비슷한 역할입니다.

## 설정 파일 위치

설정은 3가지 범위로 나뉩니다. **아래로 갈수록 우선순위가 높습니다.**

| 파일 위치 | 적용 대상 | git 커밋 |
|-----------|-----------|----------|
| `~/.claude/settings.json` | 내 모든 프로젝트 | — |
| `.claude/settings.json` | 이 프로젝트의 팀원 전체 | 커밋됨 |
| `.claude/settings.local.json` | 이 프로젝트에서 나만 | 제외됨 |

> 같은 설정이 여러 파일에 있으면 **로컬 → 프로젝트 → 사용자** 순으로 덮어씁니다.

### 언제 어떤 파일을 쓸까?

- **나만 쓸 설정** (언어, 테마 등) → `~/.claude/settings.json`
- **팀 전체가 쓸 설정** (권한 규칙, 훅 등) → `.claude/settings.json`
- **내 PC에서만 다르게** (로컬 테스트 등) → `.claude/settings.local.json`

---

## 설정 파일 열기

두 가지 방법이 있습니다.

```bash
# 방법 1: Claude Code 대화 중 명령어 입력
/config
```

```bash
# 방법 2: 에디터에서 직접 열기
code ~/.claude/settings.json
```

---

## 자주 쓰는 설정

### 기본 언어 변경

```json
{
  "language": "korean"
}
```

Claude가 한국어로 응답합니다.

### 기본 모델 지정

```json
{
  "model": "claude-sonnet-4-6"
}
```

### 환경 변수 설정

```json
{
  "env": {
    "MY_API_KEY": "your-key-here"
  }
}
```

모든 세션에 자동 적용됩니다.

---

## 권한 설정 (Permissions)

Claude가 실행할 수 있는 명령어와 읽을 수 있는 파일을 제어합니다. 집의 "출입 허용 명단"과 같습니다.

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Bash(git *)",
      "Edit(/src/**)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Edit(./config/**)"
    ]
  }
}
```

| 항목 | 동작 |
|------|------|
| `allow` | 자동 승인 — 확인 없이 바로 실행 |
| `deny` | 항상 차단 — 절대 실행 불가 |
| 규칙 없음 | 실행 전 사용자에게 확인 요청 |

### 규칙 작성법

| 규칙 | 의미 |
|------|------|
| `Bash` | 모든 Bash 명령어 |
| `Bash(npm run *)` | `npm run`으로 시작하는 명령어 |
| `Read(./.env)` | `.env` 파일 읽기 |
| `Read(./secrets/**)` | `secrets` 폴더 하위 전체 읽기 |
| `Edit(/src/**)` | `src` 폴더 하위 전체 수정 |
| `Edit(./config/**)` | `config` 폴더 하위 전체 수정 |

> `Edit` 규칙은 파일 편집과 생성을 모두 포함합니다. `Read`는 읽기만 제어합니다.

---

## 민감 파일 보호

API 키나 시크릿 파일을 Claude가 읽지 못하도록 막으려면 `deny`에 추가합니다.

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Edit(./.github/**)",
      "Edit(./infrastructure/**)"
    ]
  }
}
```

- `Read` deny — Claude가 해당 파일을 **읽지 못하도록** 차단
- `Edit` deny — Claude가 해당 파일을 **수정하지 못하도록** 차단

---

## 설정 확인

현재 적용된 설정을 확인하려면 Claude Code 대화 중:

```bash
/status
```

어떤 설정 파일이 로드되었는지, 각 설정의 출처가 함께 표시됩니다.

---

## 전체 설정 예시

실전에서 바로 쓸 수 있는 `settings.json` 예시입니다.

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "language": "korean",
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git *)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "env": {
    "NODE_ENV": "development"
  }
}
```

> `$schema`를 추가하면 VS Code 등에서 자동완성과 유효성 검사를 지원합니다.
