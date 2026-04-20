---
title: MCP 설치 후 확인하기
weight: 2
sources:
  - title: "Claude Code 공식 문서 - MCP"
    url: "https://code.claude.com/docs/en/mcp"
---

MCP 서버를 설치한 뒤 "제대로 연결됐나?"를 확인하는 방법을 정리합니다. 설치된 서버 목록 확인, `.mcp.json` 파일 구조, `settings.json`에 자동 추가되는 MCP 관련 설정까지 한눈에 볼 수 있습니다.

---

## 설치된 MCP 서버 목록 확인

### CLI에서 확인

```bash
# 전체 목록
claude mcp list

# 특정 서버 상세 정보
claude mcp get context7
```

### Claude Code 대화 중 확인

```
/mcp
```

연결 상태, 인증 여부, 사용 가능한 도구 수를 한눈에 볼 수 있습니다. 연결이 끊긴 서버는 여기서 재연결하거나 인증을 다시 할 수 있습니다.

---

## .mcp.json 확인

`--scope project`로 설치한 서버는 프로젝트 루트에 `.mcp.json` 파일로 저장됩니다.

### 파일 위치

```
프로젝트 루트/
├── .mcp.json          ← project 범위 서버
├── .claude/
│   └── settings.local.json
└── ...
```

### 구조 예시

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "x-context7-api-key": "ctx7sk-xxxxx"
      }
    }
  }
}
```

| 필드 | 설명 |
|------|------|
| `type` | 전송 방식 — `http`, `sse`, `stdio` |
| `url` | HTTP/SSE 서버 주소 |
| `headers` | 인증 헤더 (API 키 등) |
| `command` | stdio 서버의 실행 명령어 |
| `args` | stdio 서버에 전달할 인자 |
| `env` | stdio 서버에 전달할 환경변수 |

### 확인 방법

```bash
cat .mcp.json
```

> **주의**: `.mcp.json`에 API 키가 직접 들어있으면 git에 커밋할 때 노출됩니다. 팀 공유 시에는 환경변수(`${API_KEY}`)를 사용하세요.

---

## 설치 범위별 저장 위치

MCP 서버가 어디에 저장되는지는 설치할 때 지정한 `--scope`에 따라 다릅니다.

| 범위 | 저장 위치 | 팀 공유 | 확인 방법 |
|------|----------|---------|----------|
| **local** (기본) | `~/.claude.json` | 불가 | `claude mcp list` |
| **project** | `.mcp.json` | 가능 | `cat .mcp.json` |
| **user** | `~/.claude.json` | 불가 | `claude mcp list` |

- local과 user는 같은 파일(`~/.claude.json`)에 저장되지만, 적용 범위가 다릅니다.
- project는 `.mcp.json`에 저장되어 git으로 팀과 공유할 수 있습니다.

---

## settings.json에 추가되는 MCP 설정

MCP 서버를 설치하면 `settings.json`(또는 `settings.local.json`)에 **승인 관련 설정**이 자동으로 추가됩니다.

### 자동 추가되는 설정

```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": [
    "context7"
  ]
}
```

| 설정 | 의미 |
|------|------|
| `enableAllProjectMcpServers` | `.mcp.json`의 모든 project 범위 서버를 자동 승인 |
| `enabledMcpjsonServers` | 개별적으로 승인한 project 서버 목록 |

이 설정은 `.claude/settings.local.json`에 저장됩니다. project 범위 서버는 보안상 **처음 사용할 때 승인**을 받아야 하는데, 한 번 승인하면 이 설정에 기록되어 다음부터 묻지 않습니다.

### 승인 초기화

```bash
claude mcp reset-project-choices
```

이 명령을 실행하면 `enableAllProjectMcpServers`와 `enabledMcpjsonServers` 설정이 초기화되고, 다음 사용 시 다시 승인을 요청합니다.

---

## 설치 후 체크리스트

MCP 서버를 설치한 뒤 아래 항목을 순서대로 확인하세요.

| 순서 | 확인 항목 | 명령어 |
|------|----------|--------|
| 1 | 서버가 목록에 있는가 | `claude mcp list` |
| 2 | 연결 상태가 정상인가 | `/mcp` (Claude Code 내) |
| 3 | 인증이 완료되었는가 | `/mcp`에서 상태 확인 |
| 4 | project 범위라면 `.mcp.json`이 생겼는가 | `cat .mcp.json` |
| 5 | 도구가 인식되는가 | 대화에서 해당 도구 사용 시도 |

### 연결이 안 될 때

```bash
# 서버 삭제 후 재설치
claude mcp remove <서버이름>
claude mcp add --transport http <서버이름> <URL>

# 타임아웃 늘리기
MCP_TIMEOUT=10000 claude
```

---

## 활용 가이드

- **새 MCP 서버를 설치한 직후** — `/mcp`로 연결 상태를 먼저 확인하세요.
- **팀에 MCP 서버를 공유할 때** — `.mcp.json`에 API 키가 직접 포함되지 않았는지 커밋 전에 확인하세요.
- **서버가 동작하지 않을 때** — `claude mcp get <서버이름>`으로 상세 정보를 확인하고, 설정값이 올바른지 점검하세요.
