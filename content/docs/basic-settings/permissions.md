---
title: 권한 설정 (Permissions)
weight: 4
sources:
  - title: "Claude Code 공식 문서 - Permissions"
    url: "https://code.claude.com/docs/en/permissions"
---

Claude Code는 파일을 읽고, 수정하고, 명령어를 실행할 수 있습니다. 권한 설정은 이 중 **어디까지 허용할지**를 정하는 규칙입니다. 건물의 출입 카드처럼, 구역별로 "허용 / 확인 / 차단"을 지정합니다.

## 기본 동작

도구 종류에 따라 기본 승인 방식이 다릅니다.

| 종류 | 예시 | 기본 동작 |
|------|------|-----------|
| 읽기 전용 | 파일 읽기, 검색 | 자동 허용 |
| 명령어 실행 | Bash 명령어 | 매번 확인 요청 |
| 파일 수정 | 파일 편집/생성 | 매번 확인 요청 |

---

## 규칙 작성법

`settings.json`의 `permissions` 안에 3가지 규칙을 작성합니다.

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)"],
    "deny": ["Read(./.env)"],
    "ask": ["Bash(git push *)"]
  }
}
```

| 규칙 | 동작 | 비유 |
|------|------|------|
| `allow` | 확인 없이 바로 실행 | 자동문 통과 |
| `ask` | 매번 사용자에게 확인 | 안내데스크 확인 |
| `deny` | 절대 실행 불가 | 출입 차단 |

> 같은 도구에 여러 규칙이 겹치면 **deny → ask → allow** 순으로 우선 적용됩니다. deny가 항상 이깁니다.

---

## 패턴 문법

### Bash 명령어

`*`를 사용해 범위를 지정합니다.

| 패턴 | 매칭 대상 |
|------|-----------|
| `Bash` | 모든 Bash 명령어 |
| `Bash(npm run build)` | 정확히 `npm run build`만 |
| `Bash(npm run *)` | `npm run`으로 시작하는 모든 명령어 |
| `Bash(git *)` | `git`으로 시작하는 모든 명령어 |
| `Bash(* --version)` | `--version`으로 끝나는 모든 명령어 |

> `Bash(ls *)`처럼 `*` 앞에 공백이 있으면 `ls -la`는 매칭되지만 `lsof`는 매칭되지 않습니다.

### 파일 읽기/수정

gitignore 패턴을 따릅니다.

| 패턴 | 의미 |
|------|------|
| `Read(./.env)` | 현재 디렉토리의 `.env` 파일 |
| `Read(./secrets/**)` | `secrets` 폴더 하위 전체 |
| `Edit(/src/**/*.ts)` | 프로젝트 루트 기준 `src` 내 모든 `.ts` 파일 |
| `Read(~/.zshrc)` | 홈 디렉토리의 `.zshrc` |
| `Read(//tmp/data.txt)` | 절대 경로 `/tmp/data.txt` |

> `*`는 한 폴더 안에서만, `**`는 하위 폴더 전체를 매칭합니다.

### 기타 도구

| 패턴 | 매칭 대상 |
|------|-----------|
| `WebFetch(domain:example.com)` | 특정 도메인 요청 |
| `mcp__puppeteer` | puppeteer MCP 서버의 모든 도구 |
| `Agent(Explore)` | Explore 서브에이전트 |

---

## 권한 모드

Claude Code에는 여러 권한 모드가 있어, 상황에 맞게 전환할 수 있습니다.

| 모드 | 설명 | 추천 상황 |
|------|------|-----------|
| `default` | 도구 사용 시 매번 확인 | 일반적인 사용 |
| `acceptEdits` | 파일 수정은 자동 허용 | 코드 작성에 집중할 때 |
| `plan` | 분석만 가능, 수정 불가 | 코드 분석, 리팩터링 계획 |
| `auto` | AI가 안전 여부를 판단해 자동 승인 | 긴 작업, 확인 피로 줄이기 |
| `dontAsk` | 사전 허용된 도구만 실행 | 자동화 파이프라인 |
| `bypassPermissions` | 보호 경로 외 모든 권한 확인 건너뜀 | 컨테이너, VM 등 격리된 환경 |

> 각 모드의 상세 설명, 전환 방법, plan 모드의 플랜 파일 기능은 [권한 모드 (Permission Modes)]({{< relref "permission-modes" >}}) 페이지를 참고하세요.

---

## 실전 예시

### 모든 권한을 Claude Code에 넘기기

매번 "Allow" 버튼을 누르는 게 귀찮다면, 모든 도구를 자동 허용할 수 있습니다.

```json
{
  "permissions": {
    "allow": [
      "Bash",
      "Edit",
      "Write",
      "Read",
      "WebFetch",
      "Agent"
    ]
  }
}
```

| 항목 | 의미 |
|------|------|
| `Bash` | 모든 터미널 명령어 자동 실행 |
| `Edit` | 모든 파일 수정 자동 허용 |
| `Write` | 모든 파일 생성 자동 허용 |
| `Read` | 모든 파일 읽기 자동 허용 |
| `WebFetch` | 웹 요청 자동 허용 |
| `Agent` | 서브에이전트 자동 허용 |

> **주의:** 이 설정은 편리하지만, Claude가 실수로 중요한 파일을 삭제하거나 덮어쓸 수 있습니다. 개인 학습용 프로젝트처럼 **잃어버려도 괜찮은 환경**에서만 사용하세요.

---

### 코딩에 집중할 때 추천 설정

실무에서 코딩할 때 **자주 쓰는 명령어는 허용하되, 위험한 동작만 막는** 균형 잡힌 설정입니다.

```json
{
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
      "Bash(cat *)",
      "Bash(mkdir *)",
      "Bash(cp *)",
      "Bash(mv *)",
      "Bash(grep *)",
      "Bash(find *)",
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

이 설정이 하는 일:

| 구분 | 내용 |
|------|------|
| `defaultMode: acceptEdits` | 파일 수정을 확인 없이 바로 허용 |
| `allow` (빌드/실행) | npm, node, python 등 개발 명령어 자동 실행 |
| `allow` (git) | status, diff, log, add, commit 등 로컬 git 작업 허용 |
| `allow` (파일) | ls, cat, mkdir, cp, mv 등 기본 파일 작업 허용 |
| `deny` (안전장치) | `rm -rf`, `git push --force` 차단, 시크릿 파일 읽기 차단 |

> `git push`는 allow에 넣지 않았으므로 실행 전 확인을 요청합니다. 원격 저장소에 영향을 주는 명령어는 한 번 더 확인하는 것이 안전합니다.

---

### 읽기 전용 (코드 리뷰)

```json
{
  "permissions": {
    "defaultMode": "plan",
    "deny": [
      "Bash(rm *)",
      "Bash(git push *)"
    ]
  }
}
```

---

## 규칙 확인 및 관리

Claude Code 대화 중 아래 명령어로 현재 권한 상태를 확인할 수 있습니다.

```bash
/permissions
```

**Account** 탭과 **Workspace** 탭으로 나뉘어 표시됩니다.

| 탭 | 저장 위치 | 적용 범위 |
|-----|----------|-----------|
| Account | `~/.claude/settings.json` | 모든 프로젝트에 공통 적용 |
| Workspace | `.claude/settings.json` | 현재 프로젝트에만 적용 |

프로젝트마다 필요한 권한이 다를 때 Workspace 규칙이 유용합니다. 예를 들어 A 프로젝트를 B 프로젝트로 마이그레이션할 때, B 프로젝트의 Workspace에만 필요한 도구를 허용하면 다른 프로젝트에는 영향을 주지 않습니다.
