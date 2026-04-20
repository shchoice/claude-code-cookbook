---
title: 메모리 관리 (CLAUDE.md & Auto Memory)
weight: 3.3
sources:
  - title: "Claude Code 공식 문서 - Memory"
    url: "https://code.claude.com/docs/en/memory"
---

Claude Code는 매 세션 새 컨텍스트로 시작합니다. 지난 대화를 기억하지 못합니다. **메모리**는 세션을 넘어 지식을 전달하는 수단입니다.

두 가지 메모리 시스템이 있습니다:

| | CLAUDE.md | Auto Memory |
|--|-----------|-------------|
| **작성자** | 내가 직접 | Claude가 자동 |
| **내용** | 지시사항, 규칙 | 학습한 패턴, 선호도 |
| **범위** | 프로젝트/사용자/조직 | 프로젝트(워킹트리)별 |
| **용도** | 코딩 표준, 워크플로우, 아키텍처 | 빌드 명령어, 디버깅 인사이트, 발견한 선호도 |

이 문서에서는 **CLAUDE.md를 올바르게 작성하는 법**, 프로젝트가 커질수록 유용한 **`.claude/rules/` 경로별 규칙 분리**, Claude가 스스로 메모를 남기는 **Auto Memory 활용법**, 그리고 세 시스템을 어떻게 조합할지에 대한 **실전 전략**을 다룹니다.

---

## CLAUDE.md

매 세션 시작 시 자동으로 로드되는 마크다운 파일입니다. 프로젝트 규칙, 코딩 컨벤션, 빌드 명령어 등을 적어두면 Claude가 매번 읽습니다.

### 파일 위치와 범위

| 범위 | 위치 | 공유 대상 |
|------|------|---------|
| 사용자 전역 | `~/.claude/CLAUDE.md` | 나만 (모든 프로젝트) |
| 프로젝트 공유 | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 팀 전체 (git 커밋) |
| 프로젝트 개인 | `./CLAUDE.local.md` | 나만 (`.gitignore`에 추가) |

> 디렉토리 계층을 따라 올라가며 모든 CLAUDE.md를 찾아서 합칩니다. `foo/bar/`에서 실행하면 `foo/bar/CLAUDE.md`, `foo/CLAUDE.md` 모두 로드됩니다.

하위 디렉토리의 CLAUDE.md는 시작 시 로드되지 않고, Claude가 해당 디렉토리의 파일을 읽을 때 로드됩니다.

### /init으로 시작하기

```
/init
```

프로젝트를 분석해서 CLAUDE.md 초안을 자동 생성합니다. 이미 CLAUDE.md가 있으면 개선 사항을 제안합니다.

### 잘 쓰는 법

**200줄 이하로 유지하세요.** 길수록 컨텍스트를 많이 쓰고, Claude가 규칙을 무시할 확률이 높아집니다.

넣어야 할 것:

- Claude가 추측할 수 없는 빌드/테스트 명령어
- 기본과 다른 코드 스타일 규칙
- 프로젝트 아키텍처 결정
- "항상 X해라" 류의 규칙

넣지 말아야 할 것:

- 코드를 읽으면 알 수 있는 내용
- Claude가 이미 아는 표준 컨벤션
- 자주 바뀌는 정보 (링크로 대체)
- "깨끗한 코드를 작성하라" 같은 당연한 말

### 예시

```markdown
# 빌드 & 테스트
- 빌드: `npm run build`
- 테스트: `npm test -- --watch`로 단일 파일 테스트 선호
- 타입 체크: 코드 수정 후 반드시 `npx tsc --noEmit` 실행

# 코드 스타일
- ES modules (import/export) 사용, CommonJS (require) 금지
- 들여쓰기 2 스페이스
- 함수명은 camelCase, 컴포넌트명은 PascalCase

# 아키텍처
- API 핸들러: src/api/handlers/
- 비즈니스 로직: src/services/
- DB 모델: src/models/

# 워크플로우
- 커밋 메시지는 conventional commits 형식
- PR 전에 lint + test 통과 확인
```

### 다른 파일 가져오기 (@import)

`@경로` 문법으로 다른 파일을 참조할 수 있습니다.

```markdown
# CLAUDE.md
@README.md 프로젝트 개요 참고
@package.json npm 명령어 참고

# 추가 지침
- git 워크플로우: @docs/git-instructions.md
- 개인 설정: @~/.claude/my-project-instructions.md
```

> 최대 5단계까지 재귀적으로 import할 수 있습니다.

**주의 사항:**

- **상대 경로만 사용** — 절대 경로는 지원하지 않습니다.
- **코드 스팬은 임포트 처리 안 됨** — 백틱으로 감싸면 일반 텍스트로 취급됩니다.

  ```markdown
  이 줄은 임포트되지 않습니다: `@anthropic.ai/sdk`
  ```

- **중첩은 1단계까지** — 임포트된 파일 안에서의 임포트는 한 수준까지만 처리됩니다.

### AGENTS.md가 이미 있다면

다른 코딩 에이전트용 `AGENTS.md`가 있으면, CLAUDE.md에서 import합니다:

```markdown
@AGENTS.md

## Claude Code 전용
src/billing/ 하위 변경 시 plan 모드를 사용하세요.
```

---

## .claude/rules/ — 규칙 파일

CLAUDE.md가 길어지면 주제별로 분리할 수 있습니다.

### 기본 규칙

```
.claude/rules/
├── code-style.md      ← 항상 로드
├── testing.md         ← 항상 로드
└── security.md        ← 항상 로드
```

`paths` frontmatter가 없는 규칙은 매 세션 시작 시 로드됩니다.

### 경로별 규칙

특정 파일을 작업할 때만 로드되는 조건부 규칙입니다. 컨텍스트를 절약하면서 관련 지침만 전달합니다.

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 개발 규칙
- 모든 엔드포인트에 입력 유효성 검사 필수
- 표준 에러 응답 형식 사용
- OpenAPI 문서 주석 포함
```

| 패턴 | 매칭 대상 |
|------|---------|
| `**/*.ts` | 모든 디렉토리의 TypeScript 파일 |
| `src/**/*` | src 하위 모든 파일 |
| `*.md` | 프로젝트 루트의 마크다운 파일 |
| `src/components/*.tsx` | 특정 디렉토리의 React 컴포넌트 |
| `src/**/*.{ts,tsx}` | 여러 확장자를 한번에 매칭 |

> 경로별 규칙은 compaction 시 요약됩니다. 항상 유지해야 하는 규칙은 프로젝트 루트 CLAUDE.md에 넣으세요.

### 서브디렉토리 구성

규칙 수가 많아지면 서브디렉토리로 정리할 수 있습니다. 모든 `.md` 파일이 재귀적으로 탐색됩니다.

```
.claude/rules/
├── frontend/
│   ├── react.md
│   └── styles.md
├── backend/
│   ├── api.md
│   └── database.md
└── general.md
```

### 심볼릭 링크로 규칙 공유

팀이나 회사 전체에서 공통 규칙을 공유할 때 심볼릭 링크를 활용합니다.

```bash
# 공유 규칙 디렉토리 전체를 링크
ln -s ~/shared-claude-rules .claude/rules/shared

# 특정 파일만 링크
ln -s ~/company-standards/security.md .claude/rules/security.md
```

### 사용자 전역 규칙

`~/.claude/rules/`에 넣으면 모든 프로젝트에 적용됩니다.

```
~/.claude/rules/
├── preferences.md    ← 내 코딩 선호도
└── workflows.md      ← 내 워크플로우
```

> 사용자 전역 규칙은 프로젝트 규칙보다 **낮은 우선순위**로 적용됩니다. 프로젝트 규칙이 충돌 시 우선합니다.

### 모범 사례

- **주제별 파일명 사용** — `testing.md`, `api-design.md`처럼 의도가 명확한 이름을 씁니다.
- **경로별 규칙 활용** — 모든 규칙을 항상 로드하지 말고, 관련 파일 작업 시에만 로드되도록 `paths`를 설정합니다.
- **서브디렉토리로 그룹화** — 규칙이 많아지면 `frontend/`, `backend/` 등으로 분리합니다.
- **파일 하나에 하나의 주제** — 여러 주제를 섞지 않아야 로드 범위를 정확히 제어할 수 있습니다.

---

## Auto Memory

Claude가 작업하면서 **스스로 메모를 남기는** 기능입니다. 빌드 명령어, 디버깅에서 배운 점, 코드 패턴 등을 자동으로 기록합니다.

### 활성화/비활성화

기본으로 켜져 있습니다.

```
/memory → auto memory 토글
```

또는 settings.json:

```json
{
  "autoMemoryEnabled": false
}
```

### 저장 위치

```
~/.claude/projects/<프로젝트>/memory/
├── MEMORY.md              ← 인덱스 파일 (매 세션 항상 로드)
├── debugging.md           ← 디버깅 패턴 (필요 시 로드)
├── api-conventions.md     ← API 설계 결정 (필요 시 로드)
└── ...
```

**MEMORY.md**는 인덱스 역할로, 매 세션 시작 시 항상 로드됩니다 (처음 200줄 또는 25KB). 나머지 개별 파일들은 Claude가 대화 맥락상 필요하다 판단할 때만 읽습니다. 즉, Auto Memory 전체가 항상 컨텍스트를 차지하는 것이 아닙니다.

같은 git 저장소의 모든 워킹트리가 하나의 메모리 디렉토리를 공유합니다.

### 확인 및 편집

```
/memory
```

저장된 메모리 파일 목록을 보고, 에디터에서 열어 수정하거나 삭제할 수 있습니다. 모든 파일은 일반 마크다운이므로 직접 편집해도 됩니다.

### Claude에게 기억시키기

대화 중에 직접 말할 수 있습니다:

```
"pnpm만 사용하고, npm은 쓰지 마. 기억해둬"
"API 테스트는 로컬 Redis가 필요해. 기억해"
```

Claude가 auto memory에 저장합니다. CLAUDE.md에 넣고 싶다면 명시적으로 말하세요:

```
"이 규칙을 CLAUDE.md에 추가해줘"
```

---

## 조직 수준 메모리 관리

조직은 모든 사용자에게 적용되는 중앙 관리형 CLAUDE.md를 배포할 수 있습니다.

**배포 방법:**

1. **단일 저장소 방식** — 공통 CLAUDE.md를 별도 저장소에서 관리하고, 각 개발자 머신에 배포합니다.
2. **구성 관리 시스템 활용** — MDM, Group Policy, Ansible 등을 통해 `~/.claude/CLAUDE.md`를 배포합니다.

이렇게 하면 회사 전체의 보안 정책, 코딩 표준, 워크플로우 규칙을 중앙에서 일괄 관리할 수 있습니다.

---

## CLAUDE.md vs .claude/rules/ vs Auto Memory

| | CLAUDE.md | .claude/rules/ | Auto Memory |
|--|-----------|---------------|-------------|
| 작성자 | 사람 | 사람 | Claude |
| 로드 시점 | 매 세션 시작 | 시작 또는 파일 매칭 시 | MEMORY.md만 항상, 나머지는 필요 시 |
| 컨텍스트 비용 | 항상 차지 | 경로별 규칙은 조건부 | MEMORY.md만 항상, 나머지 조건부 |
| 팀 공유 | 가능 (git 커밋) | 가능 (git 커밋) | 불가 (로컬만) |
| compaction 후 | 재로드 | 경로별은 요약됨 | MEMORY.md 재로드 |

### 실전 전략

1. **CLAUDE.md**: 모든 세션에 필요한 핵심 규칙 (200줄 이하)
2. **.claude/rules/**: 특정 파일/디렉토리에만 적용되는 규칙 (경로별 분리)
3. **Auto Memory**: Claude가 알아서 기억할 것들 (수동 개입 최소화)

> CLAUDE.md가 200줄을 넘으면: 세부 워크플로우는 [Skills](https://code.claude.com/docs/en/skills)로, 경로별 규칙은 `.claude/rules/`로 분리하세요.

---

## 문제 해결

| 증상 | 원인 | 해결 |
|------|------|------|
| Claude가 CLAUDE.md 규칙을 무시 | 파일이 너무 길거나, 규칙이 모호함 | 200줄 이하로 줄이고, 구체적으로 작성 |
| 규칙 충돌 | 여러 CLAUDE.md에서 다른 지시 | `/memory`로 로드된 파일 확인, 충돌 제거 |
| `/compact` 후 규칙 사라짐 | 하위 디렉토리 CLAUDE.md는 compaction 시 요약됨 | 항상 필요한 규칙은 프로젝트 루트 CLAUDE.md에 |
| Auto memory가 뭘 저장했는지 모름 | — | `/memory`에서 확인 후 편집/삭제 |
| CLAUDE.md가 로드 안 됨 | 파일 위치가 잘못됨 | `/memory`로 로드된 파일 목록 확인 |

---

## 처음 설정한다면

1. `/init`으로 CLAUDE.md 초안을 생성
2. 빌드 명령어, 테스트 방법, 코딩 컨벤션 등 Claude가 추측할 수 없는 내용을 추가
3. Auto Memory는 켜둔 채로 사용 — Claude가 알아서 학습
4. 200줄을 넘으면 `.claude/rules/`로 분리
5. 주기적으로 `/memory`에서 auto memory 내용을 점검하고 정리
