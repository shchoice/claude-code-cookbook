---
title: MCP 서버 연결하기
weight: 1
sources:
  - title: "Claude Code 공식 문서 - MCP"
    url: "https://code.claude.com/docs/en/mcp"
---

MCP(Model Context Protocol)는 Claude Code를 외부 도구, 데이터베이스, API에 연결하는 개방형 표준입니다. 이슈 트래커, 모니터링 대시보드, DB 등의 데이터를 직접 복사해서 붙여넣는 대신, MCP 서버를 연결하면 Claude가 해당 시스템을 직접 읽고 조작할 수 있습니다.

이 문서에서는 **MCP 서버 설치와 관리**, **3가지 전송 방식(HTTP/SSE/stdio)**, **설치 범위(local/project/user)**, **OAuth 인증**, **조직 수준 관리** 방법을 다룹니다.

---

## MCP로 할 수 있는 것

MCP 서버를 연결하면 이런 작업을 Claude에게 맡길 수 있습니다:

- **이슈 트래커 연동**: "JIRA 이슈 ENG-4521의 기능을 구현하고 GitHub에 PR을 만들어줘"
- **모니터링 분석**: "Sentry에서 최근 24시간 에러를 확인해줘"
- **DB 조회**: "PostgreSQL에서 최근 90일 미접속 고객을 찾아줘"
- **워크플로우 자동화**: "이 10명에게 피드백 세션 초대 Gmail 초안을 만들어줘"
- **디자인 통합**: "Figma 디자인을 기반으로 이메일 템플릿을 수정해줘"

---

## MCP 서버 설치

3가지 전송 방식 중 선택합니다.

### HTTP 서버 (권장)

클라우드 기반 서비스에 연결할 때 사용합니다.

```bash
# 기본 문법
claude mcp add --transport http <이름> <URL>

# 예: Notion 연결
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Bearer 토큰 포함
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### SSE 서버 (비권장)

SSE(Server-Sent Events) 전송은 **deprecated** 상태입니다. 가능하면 HTTP를 사용하세요.

```bash
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

### stdio 서버

로컬 프로세스로 실행됩니다. 시스템 직접 접근이 필요한 도구에 적합합니다.

```bash
# 기본 문법
claude mcp add [옵션] <이름> -- <명령어> [인자...]

# 예: Airtable 서버
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server
```

> **옵션 순서가 중요합니다.** `--transport`, `--env`, `--scope`, `--header` 등 모든 옵션은 서버 이름 **앞**에 와야 합니다. `--`(더블 대시) 뒤에 서버에 전달할 명령어와 인자를 씁니다.

### 서버 관리 명령어

```bash
claude mcp list             # 전체 목록
claude mcp get github       # 특정 서버 상세 정보
claude mcp remove github    # 서버 삭제
```

Claude Code 안에서는 `/mcp` 명령어로 상태를 확인합니다.

---

## 설치 범위 (Scope)

서버가 어디에 저장되고, 누구와 공유되는지를 결정합니다.

| 범위 | 적용 대상 | 팀 공유 | 저장 위치 |
|------|----------|--------|----------|
| **local** (기본) | 현재 프로젝트만 | 불가 | `~/.claude.json` |
| **project** | 현재 프로젝트만 | 가능 (git 커밋) | `.mcp.json` (프로젝트 루트) |
| **user** | 모든 프로젝트 | 불가 | `~/.claude.json` |

```bash
# local (기본값)
claude mcp add --transport http stripe https://mcp.stripe.com

# project — 팀 공유용
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp

# user — 모든 프로젝트에서 사용
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### project 범위와 `.mcp.json`

`--scope project`로 추가하면 프로젝트 루트에 `.mcp.json` 파일이 생성됩니다. git에 커밋하면 팀 전체가 같은 MCP 서버를 사용합니다.

```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

보안상 `.mcp.json`의 project 범위 서버는 처음 사용 시 승인 확인을 받습니다. 승인 초기화는 `claude mcp reset-project-choices`로 합니다.

### 범위 우선순위

같은 이름의 서버가 여러 범위에 정의되면 하나만 연결됩니다:

1. Local
2. Project
3. User
4. Plugin 제공 서버
5. Claude.ai 커넥터

### `.mcp.json`에서 환경변수 사용

팀 공유 설정에서 API 키 같은 머신별 값을 처리할 때 유용합니다.

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

- `${VAR}` — 환경변수 값으로 치환
- `${VAR:-default}` — 값이 없으면 기본값 사용
- 사용 가능 위치: `command`, `args`, `env`, `url`, `headers`

---

## OAuth 인증

많은 클라우드 MCP 서버는 OAuth 2.0 인증이 필요합니다.

### 기본 흐름

```bash
# 1. 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. Claude Code 안에서 인증
/mcp
# → 브라우저에서 로그인 → 완료
```

- 토큰은 안전하게 저장되고 자동으로 갱신됩니다.
- `/mcp` 메뉴에서 "Clear authentication"으로 인증을 해제할 수 있습니다.

### 콜백 포트 고정

일부 서버는 사전 등록된 redirect URI가 필요합니다. `--callback-port`로 포트를 고정합니다.

```bash
claude mcp add --transport http \
  --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

### 사전 구성된 OAuth 자격 증명

Dynamic Client Registration을 지원하지 않는 서버는 직접 OAuth 앱을 등록해야 합니다.

```bash
claude mcp add --transport http \
  --client-id your-client-id --client-secret --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

`--client-secret` 플래그는 마스킹된 입력 프롬프트를 띄웁니다. CI 환경에서는 `MCP_CLIENT_SECRET` 환경변수로 전달합니다.

### OAuth 메타데이터 URL 오버라이드

기본 디스커버리 체인을 우회해야 할 때 `.mcp.json`에서 `authServerMetadataUrl`을 설정합니다.

```json
{
  "mcpServers": {
    "my-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "oauth": {
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

### OAuth 스코프 제한

보안 팀이 승인한 스코프만 허용하려면 `oauth.scopes`를 설정합니다.

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": {
        "scopes": "channels:read chat:write search:read"
      }
    }
  }
}
```

### 커스텀 인증 (headersHelper)

OAuth가 아닌 인증(Kerberos, 내부 SSO 등)이 필요하면 `headersHelper`로 헤더를 동적 생성합니다.

```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "/opt/bin/get-mcp-auth-headers.sh"
    }
  }
}
```

- 명령어는 JSON 객체(`{"키": "값"}`)를 stdout으로 출력해야 합니다.
- 10초 타임아웃, 매 연결마다 실행됩니다.
- `CLAUDE_CODE_MCP_SERVER_NAME`, `CLAUDE_CODE_MCP_SERVER_URL` 환경변수가 자동 설정됩니다.

---

## JSON으로 직접 추가

JSON 설정을 이미 가지고 있다면 직접 추가할 수 있습니다.

```bash
# HTTP 서버
claude mcp add-json weather-api \
  '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'

# stdio 서버
claude mcp add-json local-weather \
  '{"type":"stdio","command":"/path/to/weather-cli","args":["--api-key","abc123"]}'
```

---

## Claude Desktop에서 가져오기

Claude Desktop에 이미 MCP 서버가 설정되어 있다면 그대로 가져올 수 있습니다.

```bash
claude mcp add-from-claude-desktop
# → 어떤 서버를 가져올지 선택하는 인터랙티브 다이얼로그
```

macOS와 WSL에서만 동작합니다.

---

## Claude.ai 서버 사용

Claude.ai 계정으로 로그인했다면, [claude.ai/settings/connectors](https://claude.ai/settings/connectors)에서 추가한 MCP 서버가 Claude Code에서도 자동으로 사용 가능합니다.

비활성화하려면:

```bash
ENABLE_CLAUDEAI_MCP_SERVERS=false claude
```

---

## Claude Code를 MCP 서버로 사용

Claude Code 자체를 MCP 서버로 노출할 수 있습니다. Claude Desktop 등 다른 MCP 클라이언트에서 Claude Code의 도구(파일 읽기, 편집 등)를 사용할 수 있습니다.

```bash
claude mcp serve
```

Claude Desktop 설정 예시 (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

> `claude` 명령어가 PATH에 없으면 `which claude`로 전체 경로를 확인하여 `command`에 넣으세요.

---

## 실전 예시

### Sentry로 에러 모니터링

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

```
/mcp          ← 인증
"최근 24시간 가장 많이 발생한 에러는?"
"에러 ID abc123의 스택 트레이스를 보여줘"
```

### GitHub 코드 리뷰

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

```
/mcp          ← 인증
"PR #456을 리뷰하고 개선점을 제안해줘"
"내게 할당된 오픈 PR을 보여줘"
```

### Context7로 최신 라이브러리 문서 조회

[Context7](https://github.com/upstash/context7)은 라이브러리의 최신 문서와 코드 예시를 소스에서 직접 가져오는 MCP 서버입니다. 오래된 학습 데이터에 의존하지 않고 정확한 최신 API를 참조할 수 있습니다.

```bash
npx ctx7 setup --claude
# → OAuth 인증 후 자동 설정
```

또는 수동으로 추가:

```bash
# local (기본값 — ~/.claude.json에 저장, 나만 사용)
claude mcp add --transport http context7 https://mcp.context7.com/mcp \
  --header "x-context7-api-key: YOUR_API_KEY"

# project — .mcp.json에 저장되어 git에 커밋됨
# API 키를 직접 넣으면 팀 전체에 노출되므로 환경변수로 처리
claude mcp add --transport http context7 --scope project https://mcp.context7.com/mcp \
  --header "x-context7-api-key: ${CONTEXT7_API_KEY}"
# 팀원 각자가 CONTEXT7_API_KEY 환경변수를 본인 머신에 설정해야 합니다
```

```
"Next.js 14 미들웨어 설정 방법을 알려줘. use context7"
"Supabase 인증 API를 보여줘. use context7"
```

프롬프트에 `use context7`을 붙이면 Claude가 해당 라이브러리의 최신 공식 문서를 검색해서 답변합니다.

### PostgreSQL DB 조회

```bash
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

```
"이번 달 총 매출은?"
"orders 테이블 스키마를 보여줘"
```

---

## 추가 기능

### 동적 도구 업데이트

MCP 서버가 `list_changed` 알림을 보내면 Claude Code가 도구 목록을 자동으로 갱신합니다. 서버를 재연결할 필요가 없습니다.

### 자동 재연결

HTTP/SSE 서버가 끊기면 지수 백오프로 최대 5번 재연결을 시도합니다 (1초 → 2초 → 4초...). `/mcp`에서 상태를 확인할 수 있습니다. stdio 서버는 로컬 프로세스이므로 자동 재연결되지 않습니다.

### 채널 (Push 메시지)

MCP 서버가 `claude/channel` 기능을 선언하면 세션에 메시지를 직접 푸시할 수 있습니다. CI 결과, 모니터링 알림, 채팅 메시지 등에 Claude가 반응합니다. `--channels` 플래그로 활성화합니다.

### MCP 리소스 참조

MCP 서버가 노출하는 리소스를 `@` 멘션으로 참조할 수 있습니다.

```
@github:issue://123 분석해서 수정 방안을 제안해줘
@postgres:schema://users 와 @docs:file://database/user-model 을 비교해줘
```

### MCP 프롬프트를 명령어로 사용

MCP 서버가 노출하는 프롬프트는 `/mcp__서버명__프롬프트명` 형식의 슬래시 명령어로 사용합니다.

```
/mcp__github__list_prs
/mcp__github__pr_review 456
```

### Tool Search

MCP 서버가 많아져도 컨텍스트 부담을 줄이는 기능입니다. 세션 시작 시 도구 이름만 로드하고, Claude가 필요할 때 상세 정의를 검색합니다. 기본으로 활성화되어 있습니다.

| 환경변수 값 | 동작 |
|------------|------|
| (미설정) | 모든 MCP 도구를 지연 로드 |
| `true` | 강제로 지연 로드 활성화 |
| `auto` | 컨텍스트 10% 이내면 즉시 로드, 초과하면 지연 |
| `auto:<N>` | 커스텀 비율 (예: `auto:5`는 5%) |
| `false` | 모든 도구를 즉시 로드 |

```bash
ENABLE_TOOL_SEARCH=auto:5 claude
```

### MCP Elicitation (서버 요청 입력)

MCP 서버가 작업 중 추가 입력을 요청할 수 있습니다. 폼 모드(필드 입력) 또는 URL 모드(브라우저 인증)로 대화형 다이얼로그가 나타납니다.

### 출력 크기 제한

| 항목 | 기본값 |
|------|-------|
| 경고 임계값 | 10,000 토큰 |
| 최대 한도 | 25,000 토큰 |

```bash
# 한도 늘리기
MAX_MCP_OUTPUT_TOKENS=50000 claude
```

MCP 서버 빌더는 `_meta["anthropic/maxResultSizeChars"]`로 개별 도구의 한도를 최대 500,000자까지 설정할 수 있습니다.

---

## 조직 수준 MCP 관리

IT 관리자가 MCP 서버를 중앙에서 통제하는 2가지 방법이 있습니다.

### 방법 1: managed-mcp.json (완전 통제)

시스템 디렉토리에 배포하면, 사용자는 이 파일의 서버만 사용할 수 있습니다.

| OS | 경로 |
|----|------|
| macOS | `/Library/Application Support/ClaudeCode/managed-mcp.json` |
| Linux/WSL | `/etc/claude-code/managed-mcp.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-mcp.json` |

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "company-internal": {
      "type": "stdio",
      "command": "/usr/local/bin/company-mcp-server",
      "args": ["--config", "/etc/company/mcp-config.json"]
    }
  }
}
```

### 방법 2: 허용/차단 목록 (정책 기반)

사용자가 서버를 추가할 수 있되, 어떤 서버를 허용/차단할지 정책으로 제어합니다. 관리형 설정 파일에 `allowedMcpServers`와 `deniedMcpServers`를 사용합니다.

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem"] },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" },
    { "serverUrl": "https://*.untrusted.com/*" }
  ]
}
```

**규칙:**

- 서버 이름(`serverName`), 명령어(`serverCommand`), URL 패턴(`serverUrl`) 세 가지 기준으로 매칭합니다.
- **차단 목록이 항상 우선** — 허용 목록에 있더라도 차단 목록에 매칭되면 차단됩니다.
- 허용 목록이 빈 배열(`[]`)이면 모든 MCP 서버가 차단됩니다.
- 두 방법을 함께 사용할 수 있습니다. `managed-mcp.json`이 있으면 사용자 추가가 차단되고, 허용/차단 목록은 managed 서버 자체에 적용됩니다.

---

## 팁 모음

| 항목 | 설정 |
|------|------|
| MCP 서버 시작 타임아웃 | `MCP_TIMEOUT=10000 claude` (10초) |
| 출력 토큰 한도 | `MAX_MCP_OUTPUT_TOKENS=50000 claude` |
| Claude.ai 서버 비활성화 | `ENABLE_CLAUDEAI_MCP_SERVERS=false claude` |
| Tool Search 비활성화 | `ENABLE_TOOL_SEARCH=false claude` |
| Windows에서 npx 사용 | `claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package` |

---

## 문제 해결

| 증상 | 원인 | 해결 |
|------|------|------|
| 서버 연결 안 됨 | 타임아웃 | `MCP_TIMEOUT` 값 늘리기 |
| "Connection closed" (Windows) | npx 직접 실행 불가 | `cmd /c npx` 래퍼 사용 |
| OAuth 인증 실패 | Dynamic Client Registration 미지원 | `--client-id`, `--client-secret` 사용 |
| project 서버 동작 안 함 | 승인 미완료 | `/mcp`에서 승인, 또는 `claude mcp reset-project-choices` |
| 브라우저 리다이렉트 실패 | 콜백 URL 불일치 | `--callback-port`로 포트 고정 |
| 출력이 잘림 | 토큰 한도 초과 | `MAX_MCP_OUTPUT_TOKENS` 값 늘리기 |
