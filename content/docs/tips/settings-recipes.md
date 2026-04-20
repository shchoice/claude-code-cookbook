---
title: settings.json 실전 구성 예시
weight: 5
---

각 페이지에서 설정을 개별적으로 설명했지만, 실제로는 하나의 `settings.json`에 합쳐서 사용합니다. 이 페이지는 입문자용, 일상 코딩용, 자율 코딩용 등 **상황별 완성된 설정 파일**과 프로젝트별 설정 예시를 제공합니다. 또한 `.claude/` 디렉토리의 rules, skills, output-styles 등 각 하위 디렉토리가 어떤 역할을 하는지도 정리합니다.

> 설정 파일 위치와 범위는 [설정 파일 (Settings)]({{< relref "/docs/basic-settings/settings" >}}) 페이지를 참고하세요.

---

## 사용자 설정 (~/.claude/settings.json)

모든 프로젝트에 공통 적용되는 개인 설정입니다.

### 입문자용 — 안전하게 시작하기

처음 Claude Code를 쓰는 분에게 추천합니다. 기본 모드에서 파일 수정과 자주 쓰는 명령어만 허용합니다.

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Bash(npm *)",
      "Bash(node *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(./.env)",
      "Read(./.env.*)"
    ]
  }
}
```

### 일상 코딩용 — 균형 잡힌 설정

매일 코딩하는 개발자에게 추천합니다. 파일 수정은 자동 허용하고, 위험한 동작만 막습니다.

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "opusplan",
  "outputStyle": "Explanatory",
  "plansDirectory": ".claude/plans",
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Bash(npm *)",
      "Bash(npx *)",
      "Bash(node *)",
      "Bash(python *)",
      "Bash(pip *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git checkout *)",
      "Bash(git branch *)",
      "Bash(ls *)",
      "Bash(mkdir *)",
      "Bash(cp *)",
      "Bash(mv *)",
      "Agent"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

| 설정 | 이유 |
|------|------|
| `opusplan` | 계획은 Opus로 정확하게, 구현은 Sonnet으로 효율적으로 |
| `Explanatory` | 코드 변경 시 이유를 함께 설명받음 |
| `acceptEdits` | 파일 수정은 자동 허용. 코딩에 집중 |
| `plansDirectory` | plan 모드의 플랜 파일을 프로젝트 내에 저장 |
| `git push` 미허용 | 원격 저장소 영향 명령은 한 번 더 확인 |

### 자율 코딩용 — 최소 개입

Claude에게 자율권을 최대한 주고, 치명적 실수만 막는 설정입니다.

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "opus",
  "permissions": {
    "defaultMode": "bypassPermissions",
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

> `bypassPermissions`는 컨테이너, VM 등 **격리된 환경**에서만 사용하세요. 자세한 내용은 [권한 모드]({{< relref "/docs/basic-settings/permission-modes#bypasspermissions-모드" >}})를 참고하세요.

---

## 프로젝트 설정 (.claude/settings.json)

팀원 전체에게 적용되는 프로젝트 공통 설정입니다. git에 커밋합니다.

### 웹 프론트엔드 프로젝트

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",
      "Bash(npx *)",
      "Bash(node *)",
      "Bash(git *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./.env.local)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

| 설정 | 이유 |
|------|------|
| `hooks.PostToolUse` | 파일 수정 후 자동으로 prettier 포맷팅 |
| `.env.local` deny | Next.js 등에서 쓰는 로컬 환경변수 보호 |

### Python 백엔드 프로젝트

```json
{
  "permissions": {
    "allow": [
      "Bash(python *)",
      "Bash(pip *)",
      "Bash(pytest *)",
      "Bash(uv *)",
      "Bash(ruff *)",
      "Bash(mypy *)",
      "Bash(git *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Edit(./alembic/versions/**)"
    ]
  }
}
```

| 설정 | 이유 |
|------|------|
| `alembic/versions` deny | DB 마이그레이션 파일은 수동 관리 |
| `ruff`, `mypy` allow | 린트와 타입 체크 자동 실행 허용 |

---

## 로컬 설정 (.claude/settings.local.json)

나만 쓰는 프로젝트 수준 설정입니다. `.gitignore`에 포함되어 커밋되지 않습니다.

```json
{
  "model": "opus",
  "outputStyle": "Korean Dev",
  "permissions": {
    "defaultMode": "acceptEdits"
  },
  "env": {
    "DATABASE_URL": "postgresql://localhost:5432/mydb_dev"
  }
}
```

> 프로젝트 설정에서 `sonnet`을 기본 모델로 지정했더라도, 로컬 설정에서 `opus`로 덮어쓸 수 있습니다. 로컬이 항상 우선입니다.

---

## settings.json이 관리하는 마크다운 파일들

`settings.json`의 일부 설정은 **별도의 마크다운 파일**과 연결됩니다. 설정 파일은 "어디를 보라"만 지정하고, 실제 내용은 `.md` 파일에 작성합니다.

### plansDirectory — 플랜 파일

plan 모드에서 Claude가 작성하는 계획서가 저장되는 위치입니다.

```json
{
  "plansDirectory": ".claude/plans"
}
```

| 설정값 | 저장 위치 | 특징 |
|--------|----------|------|
| 미설정 (기본) | `~/.claude/plans/` | 개인 디렉토리. 프로젝트 간 공유 안 됨 |
| `".claude/plans"` | 프로젝트 `.claude/plans/` | git 커밋 가능. 팀과 공유 가능 |
| `"./plans"` | 프로젝트 루트 `plans/` | 문서처럼 관리할 때 |

플랜 파일은 Claude가 자동 생성하며, 파일명도 자동으로 붙습니다. 디렉토리 구조 예시:

```
.claude/plans/
├── 2026-04-15-oauth-migration.md
├── 2026-04-18-api-refactor.md
└── 2026-04-20-test-coverage.md
```

각 파일에는 Claude가 분석한 현재 상태, 변경 계획, 파일 목록 등이 담깁니다. `Ctrl + G`로 에디터에서 직접 수정할 수 있고, 플랜을 승인하면 Claude가 이 계획에 따라 구현합니다.

> 프로젝트에 `.claude/plans`를 사용하면 "왜 이렇게 구현했는지" 기록이 남아 나중에 참고할 수 있습니다.

### outputStyle — 커스텀 출력 스타일

커스텀 출력 스타일은 마크다운 파일로 작성하고, `settings.json`에서 이름으로 참조합니다.

```json
{
  "outputStyle": "Korean Dev"
}
```

| 저장 위치 | 적용 범위 |
|----------|---------|
| `~/.claude/output-styles/` | 모든 프로젝트 |
| `.claude/output-styles/` | 현재 프로젝트만 |

디렉토리 구조 예시:

```
~/.claude/output-styles/
├── korean-dev.md
└── code-reviewer.md

.claude/output-styles/
└── team-style.md
```

각 파일은 **frontmatter + 지침**으로 구성됩니다:

```markdown
---
name: Korean Dev
description: 한국어로 응답하고 코드 설명을 덧붙이는 스타일
keep-coding-instructions: true
---

# 응답 지침

- 모든 응답과 커밋 메시지를 한국어로 작성합니다
- 코드 변경 시 변경 이유를 한 줄로 설명합니다
```

| frontmatter | 설명 |
|-------------|------|
| `name` | `/config`에 표시되는 이름. `outputStyle` 값과 매칭 |
| `description` | 스타일 선택 시 보이는 설명 |
| `keep-coding-instructions` | `true`면 기본 코딩 지침 유지, `false`면 코딩 지침 제거 |

> 기본 제공 스타일(`Default`, `Explanatory`, `Learning`)은 파일이 없어도 이름만으로 사용할 수 있습니다. 커스텀 스타일만 `.md` 파일이 필요합니다.

### .claude/ 디렉토리 구조 이해하기

`.claude/` 아래 디렉토리는 각각 **다른 목적**을 가집니다. 겉보기엔 비슷한 마크다운 파일이지만, 어디에 놓느냐에 따라 **로드 시점과 역할이 완전히 다릅니다.**

```
.claude/
├── settings.json        ← 설정값 (JSON)
├── settings.local.json  ← 개인 설정값 (JSON)
├── CLAUDE.md            ← 프로젝트 지침
│
├── rules/               ← "이렇게 해라" 규칙
├── output-styles/       ← "이런 방식으로 말해라" 응답 스타일
├── skills/              ← "이 절차대로 수행해라" 작업 템플릿
├── agents/              ← 서브에이전트 정의
├── plans/               ← plan 모드 결과물
└── worktrees/           ← git worktree 격리 공간
```

#### 핵심 구분: rules vs skills vs output-styles

이 세 디렉토리가 가장 헷갈립니다. 비유로 정리하면:

| 디렉토리 | 비유 | 역할 | 로드 시점 |
|---------|------|------|---------|
| `rules/` | 회사 코딩 가이드북 | "변수명은 camelCase로" 같은 **규칙** | 세션 시작 시 자동 (경로별은 파일 매칭 시) |
| `skills/` | 업무 매뉴얼 | "1단계: 테스트 → 2단계: 빌드 → 3단계: 배포" 같은 **절차** | `/skill-name`으로 호출하거나 Claude가 자동 판단 |
| `output-styles/` | 말투 설정 | "한국어로 친절하게 설명하듯이" 같은 **응답 방식** | `/config`에서 선택하면 항상 |

**같은 주제라도 디렉토리에 따라 용도가 다릅니다:**

| 주제 | rules/에 넣으면 | skills/에 넣으면 |
|------|---------------|----------------|
| 테스트 | "mock 대신 실제 DB를 써라" (규칙) | "테스트 실행 → 실패 분석 → 수정 → 재실행" (절차) |
| 보안 | "사용자 입력은 반드시 sanitize" (규칙) | "변경사항의 보안 취약점을 분석하고 리포트 작성" (절차) |
| 코드 스타일 | "2 스페이스 들여쓰기" (규칙) | 해당 없음 (스타일 규칙은 rules/에) |

#### 컨텍스트 비용 차이

| 디렉토리 | 컨텍스트 차지 | 비고 |
|---------|------------|------|
| `rules/` (기본) | **항상** 차지 | 매 세션 로드 |
| `rules/` (경로별) | **조건부** | 해당 파일 작업 시만 |
| `skills/` | **호출 시만** | 평소엔 설명 한 줄만 로드 |
| `output-styles/` | **항상** 차지 | 선택된 스타일만 |

> rules/에 모든 걸 넣으면 매 세션 컨텍스트를 차지합니다. 자주 안 쓰는 워크플로우는 skills/로 분리하세요.

#### 전체 구조 예시

```
~/.claude/                              ← 사용자 전역
├── settings.json
├── CLAUDE.md
├── rules/
│   └── preferences.md                  ← 내 코딩 선호도 (항상 로드)
├── output-styles/
│   └── korean-dev.md                   ← 커스텀 출력 스타일
└── plans/                              ← 기본 플랜 저장 위치

프로젝트/
├── CLAUDE.md                           ← 프로젝트 지침 (항상 로드)
├── .claude/
│   ├── settings.json                   ← 프로젝트 설정 (팀)
│   ├── settings.local.json             ← 로컬 설정 (나만)
│   ├── rules/
│   │   ├── code-style.md               ← 항상 로드되는 규칙
│   │   ├── testing.md                  ← 항상 로드되는 규칙
│   │   └── api-conventions.md          ← paths: "src/api/**" → 조건부 로드
│   ├── output-styles/
│   │   └── team-style.md               ← 프로젝트 출력 스타일
│   ├── skills/
│   │   ├── fix-issue/SKILL.md          ← /fix-issue로 호출
│   │   └── deploy/SKILL.md             ← /deploy로 호출
│   ├── agents/
│   │   └── security-reviewer.md        ← 서브에이전트
│   └── plans/
│       └── 2026-04-20-auth-migration.md ← plan 모드 결과물
```

---

## 설정 우선순위 정리

같은 키가 여러 파일에 있을 때 적용 순서입니다.

```
.claude/settings.local.json  ← 최우선 (나만)
.claude/settings.json         ← 프로젝트 (팀)
~/.claude/settings.json       ← 사용자 (전역)
```

> **deny 규칙은 특별합니다.** 어떤 수준에서 deny하면 다른 수준에서 allow할 수 없습니다. deny는 항상 이깁니다.

---

## 설정 항목 빠른 참조

이 쿡북에서 다룬 주요 설정 항목입니다.

| 설정 키 | 값 예시 | 설명 | 관련 페이지 |
|---------|--------|------|-----------|
| `model` | `"opusplan"` | 기본 모델 | [모델 설정]({{< relref "/docs/basic-settings/model-config" >}}) |
| `outputStyle` | `"Explanatory"` | 출력 스타일 | [출력 스타일]({{< relref "/docs/basic-settings/output-styles" >}}) |
| `plansDirectory` | `".claude/plans"` | 플랜 파일 저장 위치 | [권한 모드]({{< relref "/docs/basic-settings/permission-modes" >}}) |
| `permissions.defaultMode` | `"acceptEdits"` | 기본 권한 모드 | [권한 모드]({{< relref "/docs/basic-settings/permission-modes" >}}) |
| `permissions.allow` | `["Bash(npm *)"]` | 자동 허용 규칙 | [권한 설정]({{< relref "/docs/basic-settings/permissions" >}}) |
| `permissions.deny` | `["Read(./.env)"]` | 차단 규칙 | [권한 설정]({{< relref "/docs/basic-settings/permissions" >}}) |
| `statusLine` | `{"type":"command",...}` | 상태 표시줄 스크립트 | [상태 표시줄]({{< relref "/docs/tips/statusline" >}}) |
| `hooks` | `{"PostToolUse":[...]}` | 이벤트 훅 | [공식 문서](https://code.claude.com/docs/en/hooks-guide) |
| `env` | `{"NODE_ENV":"dev"}` | 환경변수 | [설정 파일]({{< relref "/docs/basic-settings/settings" >}}) |
| `language` | `"korean"` | 응답 언어 | [설정 파일]({{< relref "/docs/basic-settings/settings" >}}) |
| `effortLevel` | `"high"` | 사고 깊이 | [모델 설정]({{< relref "/docs/basic-settings/model-config" >}}) |
