---
title: 권한 모드 (Permission Modes)
weight: 5
sources:
  - title: "Claude Code 공식 문서 - Permission Modes"
    url: "https://code.claude.com/docs/en/permission-modes"
  - title: "Claude Code 공식 문서 - Common Workflows (Plan Mode)"
    url: "https://code.claude.com/docs/en/common-workflows#how-to-use-plan-mode"
---

권한 모드는 Claude Code가 **얼마나 자주 허락을 구할지**를 결정합니다. 모드에 따라 매 동작마다 확인을 받을 수도, 끊김 없이 자율적으로 작업할 수도 있습니다.

## 모드 전환 방법

### Shift + Tab (대화 중 전환)

대화 도중 `Shift + Tab`을 누르면 모드가 순환합니다.

```
default → acceptEdits → plan → (bypassPermissions) → (auto)
```

`bypassPermissions`와 `auto`는 괄호로 표시한 것처럼, 특정 조건을 만족해야 순환 목록에 나타납니다.

| 모드 | 순환 목록에 나타나는 조건 |
|------|--------------------------|
| `bypassPermissions` | `--permission-mode bypassPermissions` 또는 `--allow-dangerously-skip-permissions`로 시작한 세션 |
| `auto` | 계정 요건 충족 시 (Max, Team, Enterprise, API 플랜) |

### CLI 플래그 (시작 시 지정)

```bash
claude --permission-mode plan
```

### settings.json (기본값 고정)

```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

---

## 모드 한눈에 보기

| 모드 | 확인 없이 실행되는 것 | 추천 상황 |
|------|----------------------|-----------|
| `default` | 파일 읽기만 | 처음 사용할 때, 민감한 작업 |
| `acceptEdits` | 파일 읽기 + 수정 + 기본 파일 명령어 | 코딩에 집중할 때 |
| `plan` | 파일 읽기만 (수정 불가) | 코드 분석, 리팩터링 계획 |
| `auto` | 전부 (백그라운드 안전 검사 있음) | 긴 작업, 확인 피로 줄이기 |
| `dontAsk` | 사전 허용된 도구만 | CI/CD 파이프라인 |
| `bypassPermissions` | 보호 경로 외 전부 | 컨테이너, VM 등 격리 환경 |

---

## 모드별 상세

### default

가장 기본적인 모드입니다. 파일 읽기만 자동 허용되고, 나머지는 매번 확인합니다.

**언제 쓸까:** Claude Code를 처음 써보거나, 운영 서버 코드를 다룰 때.

### acceptEdits

파일 수정과 기본 파일 명령어(`mkdir`, `touch`, `rm`, `mv`, `cp`, `sed`)를 확인 없이 실행합니다. 작업 디렉토리 내 경로에만 적용됩니다.

**언제 쓸까:** 코드를 작성하면서 `git diff`로 나중에 한꺼번에 확인하고 싶을 때.

```bash
claude --permission-mode acceptEdits
```

### plan

Claude가 코드를 **분석하고 계획만 세우는** 모드입니다. 파일을 수정하거나 위험한 명령어를 실행하지 않습니다.

**언제 쓸까:**

- 여러 파일을 건드리는 큰 기능을 구현하기 전에 방향을 잡을 때
- 낯선 코드베이스를 파악할 때
- Claude와 대화하며 구현 방향을 조율할 때

자세한 내용은 아래 [plan 모드 심화](#plan-모드-심화) 참고.

### auto

모든 도구를 확인 없이 실행하되, **별도의 분류 모델이 백그라운드에서 안전 여부를 검사**합니다. 위험하다고 판단되면 자동으로 차단됩니다.

**언제 쓸까:** 방향이 명확한 긴 작업을 맡기고, 중간에 확인 클릭으로 흐름이 끊기는 걸 원하지 않을 때.

사용 조건:

| 조건 | 요구사항 |
|------|----------|
| 플랜 | Max, Team, Enterprise, API (Pro 불가) |
| 모델 | Sonnet 4.6, Opus 4.6, Opus 4.7 |
| 프로바이더 | Anthropic API만 (Bedrock, Vertex 불가) |

> `bypassPermissions`와 달리 안전 검사가 있으므로, 격리되지 않은 환경에서도 사용할 수 있습니다.

### dontAsk

`permissions.allow`에 사전 등록한 도구만 실행하고, 나머지는 확인 없이 **거부**합니다. 사람이 개입할 수 없는 환경을 위한 모드입니다.

**언제 쓸까:** CI/CD 파이프라인이나 스크립트에서 Claude Code를 실행할 때.

```bash
claude --permission-mode dontAsk
```

### bypassPermissions

모든 권한 확인을 건너뜁니다. [보호 경로](#보호-경로)에 쓰기할 때만 예외적으로 확인합니다.

**언제 쓸까:** 컨테이너, VM, devcontainer 등 **Claude가 망가뜨려도 괜찮은 격리 환경**에서만 사용하세요.

```bash
claude --permission-mode bypassPermissions
```

> 관리자가 `permissions.disableBypassPermissionsMode`를 `"disable"`로 설정하면 이 모드를 차단할 수 있습니다.

---

## plan 모드 심화

### 기본 사용법

plan 모드로 진입하면 Claude는 코드를 읽고 분석한 뒤, **플랜 파일(.md)**을 작성합니다.

```bash
claude --permission-mode plan
```

```
OAuth2로 인증 시스템을 마이그레이션하는 계획을 세워줘
```

Claude가 현재 구현을 분석하고 마이그레이션 계획을 작성합니다. 후속 질문으로 계획을 다듬을 수 있습니다.

```
하위 호환성은 어떻게 처리할까?
```

### 플랜 파일 저장 위치

플랜 파일은 기본적으로 `~/.claude/plans`에 저장됩니다. 프로젝트별로 관리하고 싶다면 `settings.json`에서 경로를 변경할 수 있습니다.

```json
{
  "plansDirectory": ".claude/plans"
}
```

| 설정 | 저장 위치 | 용도 |
|------|-----------|------|
| 기본값 | `~/.claude/plans` | 개인 플랜, 프로젝트 간 공유 없음 |
| `".claude/plans"` | 프로젝트 내 `.claude/plans/` | 팀과 공유하거나 git에 커밋 |
| `"./plans"` | 프로젝트 루트의 `plans/` | 프로젝트 문서로 관리 |

### 플랜 편집과 승인

`Ctrl + G`를 누르면 플랜 파일이 기본 텍스트 에디터에서 열립니다. Claude가 작성한 계획을 직접 수정한 뒤 저장하면 반영됩니다.

플랜이 준비되면 Claude가 실행 방식을 물어봅니다.

| 선택지 | 동작 |
|--------|------|
| auto 모드로 실행 | 계획을 auto 모드로 바로 구현 |
| acceptEdits로 실행 | 파일 수정은 자동, 나머지는 확인 |
| 매번 확인하며 실행 | default 모드로 한 단계씩 진행 |
| 계속 계획 수정 | 피드백을 주고 계획을 더 다듬기 |

> 플랜을 승인하면 세션 이름이 플랜 내용에서 자동으로 지정됩니다. `--name`이나 `/rename`으로 이미 이름을 지정한 경우에는 덮어쓰지 않습니다.

---

## 보호 경로

어떤 모드에서든 아래 경로에 쓰기할 때는 추가 확인이 필요합니다.

**디렉토리:**

| 경로 | 보호 이유 |
|------|-----------|
| `.git` | 저장소 상태 손상 방지 |
| `.claude` | Claude Code 설정 보호 |
| `.vscode` | 에디터 설정 보호 |
| `.idea` | 에디터 설정 보호 |
| `.husky` | git 훅 보호 |

> `.claude/commands`, `.claude/agents`, `.claude/skills`, `.claude/worktrees`는 예외입니다. Claude가 자주 쓰는 경로이므로 확인 없이 쓰기 가능합니다.

**파일:**

`.gitconfig`, `.gitmodules`, `.bashrc`, `.bash_profile`, `.zshrc`, `.zprofile`, `.profile`, `.ripgreprc`, `.mcp.json`, `.claude.json`
