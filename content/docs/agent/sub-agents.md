---
title: 서브에이전트(Subagent) 만들고 활용하기
weight: 1
sources:
  - title: "Claude Code 공식 문서 - Subagents"
    url: "https://code.claude.com/docs/en/sub-agents"
  - title: "Claude Code 공식 문서 - Features Overview"
    url: "https://code.claude.com/docs/en/features-overview"
---

서브에이전트(Subagent)는 Claude Code가 특정 작업을 위임하는 **전문 어시스턴트**입니다. 자체 컨텍스트 윈도우, 자체 시스템 프롬프트, 자체 도구 권한을 가집니다. 메인 대화는 깔끔하게 유지하면서 잡다한 탐색이나 검증은 서브에이전트에게 맡길 수 있습니다.

이 문서에서는 **서브에이전트의 개념**, **내장 서브에이전트**, **커스텀 서브에이전트 만들기**, **호출 방법**, **실전 예시**까지 다룹니다.

---

## 비유: 팀장과 전문 직원

메인 Claude Code를 **팀장**이라고 생각해 보세요. 팀장이 모든 일을 직접 하는 대신, 특정 분야 전문 직원에게 일을 맡길 수 있습니다.

| 메인 Claude Code (팀장) | 서브에이전트 (전문 직원) |
|------------------------|------------------------|
| 사용자와 직접 대화 | 팀장의 지시로 작업 |
| 전체 흐름 조율 | 자기 분야만 집중 |
| 모든 컨텍스트를 봐야 함 | 자기 작업에 필요한 것만 봄 |
| 결과물을 사용자에게 전달 | 결과를 팀장에게 보고 |

코드 리뷰 전문 직원에게 "이 PR 리뷰해줘"라고 시키면, 직원이 파일 수십 개를 읽고 분석한 뒤 **요약된 결과만** 팀장 책상에 올려놓습니다. 팀장은 잡다한 중간 과정을 신경 쓸 필요가 없습니다.

---

## 언제 서브에이전트를 쓰나?

| 상황 | 이유 |
|------|------|
| 출력이 큰 작업 (테스트 실행, 로그 분석) | 메인 대화 컨텍스트 보호 |
| 도구 제한이 필요한 작업 | "쓰기는 안 됨" 등 강제 가능 |
| 같은 작업을 반복할 때 | 한 번 만들어두고 재사용 |
| 빠른 모델로 처리하고 싶을 때 | Haiku로 비용 절감 |
| 병렬 리서치 | 여러 영역을 동시에 탐색 |

> 같은 종류의 워커를 같은 지시로 계속 띄우고 있다면 → 커스텀 서브에이전트로 만들 타이밍입니다.

---

## 내장 서브에이전트

Claude Code는 기본 서브에이전트 몇 개를 내장하고 있습니다. 직접 만들지 않아도 자동으로 사용됩니다.

| 이름 | 모델 | 역할 |
|------|------|------|
| **Explore** | Haiku | 코드베이스 탐색/검색 (읽기 전용) |
| **Plan** | 메인 상속 | Plan 모드에서 코드베이스 리서치 |
| **general-purpose** | 메인 상속 | 복잡한 다단계 작업 (탐색+수정) |
| **statusline-setup** | Sonnet | `/statusline` 설정 도우미 |

Explore는 Claude가 **검색이 필요할 때 자동으로** 위임합니다. 검색 결과가 메인 대화에 쌓이지 않게 됩니다.

---

## 커스텀 서브에이전트 만들기

3가지 방법이 있습니다.

### 방법 1: `/agents` 명령어 (권장)

대화형 인터페이스로 단계별로 만들 수 있습니다.

```
/agents
```

Library 탭에서 **Create new agent** → **Personal** 선택. Claude가 description, system prompt를 자동 생성해줍니다. 도구 선택, 모델 선택, 색상 지정까지 가이드가 따라옵니다.

### 방법 2: 파일로 직접 작성

서브에이전트는 **YAML frontmatter + Markdown 본문** 형식의 파일입니다.

```markdown
---
name: code-reviewer
description: 코드 품질과 보안을 검토합니다. 코드 변경 후 적극적으로 사용하세요.
tools: Read, Grep, Glob, Bash
model: sonnet
---

당신은 시니어 코드 리뷰어입니다. 호출되면:

1. `git diff`로 최근 변경사항 확인
2. 수정된 파일에 집중
3. 즉시 리뷰 시작

체크리스트:
- 코드가 명확하고 읽기 쉬운가
- 함수/변수 이름이 적절한가
- 중복 코드는 없는가
- 에러 처리가 적절한가
- 비밀키가 노출되어 있지 않은가
```

### 방법 3: `--agents` CLI 플래그

세션에 일회성으로 정의합니다. 디스크에 저장되지 않습니다.

```bash
claude --agents '{
  "code-reviewer": {
    "description": "코드 변경 후 적극적으로 사용하세요.",
    "prompt": "당신은 시니어 코드 리뷰어입니다...",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

---

## 저장 위치 (Scope)

같은 이름의 서브에이전트가 여러 위치에 있으면 우선순위가 높은 것이 적용됩니다.

| 위치 | 적용 범위 | 우선순위 | 팀 공유 |
|------|----------|---------|---------|
| Managed settings | 조직 전체 | 1 (최고) | - |
| `--agents` CLI 플래그 | 현재 세션만 | 2 | - |
| `.claude/agents/` | 현재 프로젝트 | 3 | 가능 (git 커밋) |
| `~/.claude/agents/` | 모든 프로젝트 | 4 | 불가 |
| 플러그인 `agents/` | 플러그인 활성화 시 | 5 (최저) | - |

> **팀 공유** — `.claude/agents/`에 두고 git에 커밋하면 팀 전체가 같은 서브에이전트를 쓸 수 있습니다.

---

## frontmatter 필드

`name`과 `description`만 필수입니다.

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | O | 소문자/하이픈으로 된 고유 식별자 |
| `description` | O | Claude가 위임 여부를 판단하는 기준 |
| `tools` | X | 허용 도구 목록 (생략 시 메인 도구 모두 상속) |
| `disallowedTools` | X | 차단 도구 목록 |
| `model` | X | `sonnet`, `opus`, `haiku`, `inherit` (기본값) |
| `permissionMode` | X | 권한 모드 (`default`, `acceptEdits`, `plan` 등) |
| `maxTurns` | X | 최대 턴 수 |
| `skills` | X | 시작 시 주입할 Skill 목록 |
| `mcpServers` | X | 이 서브에이전트만 쓸 MCP 서버 |
| `hooks` | X | 라이프사이클 훅 |
| `memory` | X | 영속 메모리 (`user`/`project`/`local`) |
| `isolation` | X | `worktree`로 설정하면 격리된 git worktree에서 실행 |
| `color` | X | 표시 색상 |

---

## 도구 제한

서브에이전트가 할 수 있는 일을 제한합니다.

### `tools` — 허용 목록

```yaml
---
name: safe-researcher
description: 제한된 권한의 리서치 에이전트
tools: Read, Grep, Glob, Bash
---
```

이 서브에이전트는 파일 수정, 쓰기, MCP 도구 모두 사용 불가.

### `disallowedTools` — 차단 목록

```yaml
---
name: no-writes
description: 쓰기 도구만 빼고 모든 도구 사용 가능
disallowedTools: Write, Edit
---
```

메인 대화의 모든 도구를 상속하되 `Write`, `Edit`만 제외.

> 두 필드를 함께 쓰면 `disallowedTools`가 먼저 적용되고 `tools`는 남은 풀에서 골라집니다.

---

## 권한 모드

`permissionMode`로 권한 프롬프트 동작을 제어합니다.

| 모드 | 동작 |
|------|------|
| `default` | 기본 권한 체크 (프롬프트로 확인) |
| `acceptEdits` | 작업 디렉토리 내 편집 자동 승인 |
| `auto` | 백그라운드 분류기가 명령을 검토 |
| `dontAsk` | 권한 프롬프트 자동 거부 |
| `bypassPermissions` | 모든 권한 프롬프트 건너뛰기 (주의) |
| `plan` | 읽기 전용 탐색 모드 |

> 부모 세션이 `bypassPermissions`나 `acceptEdits`면 서브에이전트가 이를 덮어쓸 수 없습니다.

---

## 호출 방법

### 자동 위임

Claude가 `description`을 보고 알아서 위임합니다. description에 **"use proactively"**, **"코드 변경 후 적극적으로 사용"** 같은 문구를 넣으면 적극적으로 호출됩니다.

### 자연어 지정

```
code-reviewer 서브에이전트로 최근 변경사항 리뷰해줘
test-runner 서브에이전트를 써서 실패한 테스트 고쳐줘
```

### `@`-멘션 (가장 확실)

`@`를 입력하면 등록된 서브에이전트 목록이 뜹니다.

```
@code-reviewer auth 변경사항 봐줘
```

특정 서브에이전트가 **반드시** 실행됩니다.

### 세션 전체로 사용 (`--agent`)

세션 자체를 서브에이전트로 시작합니다.

```bash
claude --agent code-reviewer
```

서브에이전트의 시스템 프롬프트가 기본 Claude Code 프롬프트를 **완전히 대체**합니다.

프로젝트 기본값으로 두려면 `.claude/settings.json`:

```json
{
  "agent": "code-reviewer"
}
```

---

## 포그라운드 vs 백그라운드

| | 포그라운드 | 백그라운드 |
|--|----------|-----------|
| 메인 대화 | 완료까지 대기 | 동시에 작업 가능 |
| 권한 프롬프트 | 사용자에게 전달됨 | 사전 승인된 것만 허용 |
| 사용 시점 | 결과를 바로 봐야 할 때 | 시간 걸리는 작업 |

`Ctrl+B`를 누르면 실행 중인 작업을 백그라운드로 보낼 수 있습니다.

---

## 영속 메모리 (`memory`)

서브에이전트가 **여러 세션에 걸쳐 학습**하게 합니다.

```yaml
---
name: code-reviewer
description: 코드 품질과 보안을 검토합니다.
memory: project
---

리뷰하면서 발견한 패턴, 컨벤션, 반복되는 문제는 메모리에 기록하세요.
```

| 스코프 | 위치 | 용도 |
|-------|------|------|
| `user` | `~/.claude/agent-memory/<이름>/` | 모든 프로젝트에서 학습 공유 |
| `project` | `.claude/agent-memory/<이름>/` | 프로젝트 한정, git 공유 가능 |
| `local` | `.claude/agent-memory-local/<이름>/` | 프로젝트 한정, git 미공유 |

서브에이전트는 매 세션마다 메모리를 읽고, 새로 배운 것을 추가합니다. 시간이 지날수록 더 똑똑해집니다.

---

## 실전 예시

### 코드 리뷰어 (읽기 전용)

```markdown
---
name: code-reviewer
description: 코드 품질과 보안 전문 리뷰어. 코드 변경 후 적극적으로 사용하세요.
tools: Read, Grep, Glob, Bash
model: inherit
---

당신은 시니어 코드 리뷰어입니다.

호출되면:
1. `git diff`로 최근 변경 확인
2. 수정된 파일에 집중
3. 즉시 리뷰 시작

체크 항목:
- 가독성, 명명 규칙
- 중복 코드, 에러 처리
- 비밀키/API 키 노출 여부
- 입력 검증
- 테스트 커버리지

피드백은 우선순위로 정리:
- Critical (반드시 수정)
- Warning (수정 권장)
- Suggestion (개선 고려)

각 이슈에 구체적 수정 예시를 포함하세요.
```

### 디버거 (수정 가능)

```markdown
---
name: debugger
description: 에러, 테스트 실패, 예상치 못한 동작 전문 디버거. 문제 발생 시 적극적으로 사용하세요.
tools: Read, Edit, Bash, Grep, Glob
---

당신은 근본 원인 분석 전문가입니다.

호출되면:
1. 에러 메시지/스택 트레이스 캡처
2. 재현 방법 식별
3. 실패 위치 격리
4. 최소 수정 적용
5. 해결 검증

각 이슈에 대해:
- 근본 원인 설명
- 진단 근거
- 구체적 수정 코드
- 테스트 방법

증상이 아니라 근본 원인을 고치세요.
```

### 읽기 전용 DB 쿼리

```markdown
---
name: db-reader
description: 읽기 전용 DB 쿼리 실행. 데이터 분석/리포트 생성 시 사용.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

당신은 읽기 전용 DB 분석가입니다. SELECT 쿼리만 실행할 수 있습니다.

INSERT/UPDATE/DELETE 요청을 받으면 거부하고 읽기 전용임을 설명하세요.
```

검증 스크립트 (`./scripts/validate-readonly-query.sh`):

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE)\b' > /dev/null; then
  echo "차단됨: SELECT 쿼리만 허용됩니다." >&2
  exit 2
fi

exit 0
```

훅이 exit code 2로 종료하면 명령이 차단되고 Claude에게 에러가 전달됩니다.

---

## 설정 확인 명령어

```bash
# 등록된 모든 서브에이전트 목록
claude agents

# 대화 중 관리 인터페이스
/agents

# 실행 중인 서브에이전트 확인
/agents → Running 탭
```

---

## 알아두면 좋은 점

| 항목 | 내용 |
|------|------|
| 컨텍스트 격리 | 서브에이전트는 메인 대화 기록을 상속받지 않음 |
| 작업 디렉토리 | 메인 대화의 cwd에서 시작. `cd`는 호출 간 유지 안 됨 |
| 중첩 불가 | 서브에이전트가 다른 서브에이전트를 띄울 수 없음 |
| Skills 상속 | 메인의 Skill을 자동 상속하지 않음 — `skills:` 필드로 명시 필요 |
| 트랜스크립트 | `~/.claude/projects/{프로젝트}/{세션ID}/subagents/`에 별도 저장 |

---

## 활용 가이드

- **반복 작업이 보이면 서브에이전트로 만드세요.** 같은 지시를 두 번 이상 했다면 후보입니다.
- **description에 "적극적으로 사용" 같은 문구를 넣으세요.** Claude가 알아서 위임할 확률이 올라갑니다.
- **읽기 전용 작업은 도구를 제한하세요.** `tools: Read, Grep, Glob, Bash`로 두면 실수로 파일이 수정될 일이 없습니다.
- **출력이 큰 작업은 무조건 서브에이전트로.** 테스트 실행, 로그 분석은 메인 대화를 오염시키지 않습니다.
- **여러 영역을 병렬로 봐야 한다면** 서브에이전트 여러 개를 동시에 띄우세요. 단, 결과 요약이 메인에 다 돌아오므로 컨텍스트가 부담되면 [에이전트 팀](../agent-teams)을 고려하세요.
