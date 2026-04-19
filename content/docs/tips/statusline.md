---
title: 상태 표시줄 (Status Line)
weight: 3
sources:
  - title: "Claude Code 공식 문서 - Status Line"
    url: "https://code.claude.com/docs/en/statusline"
---

Status Line은 Claude Code 하단에 표시되는 커스터마이징 가능한 상태 바입니다. 모델명, 컨텍스트 사용량, 비용, git 브랜치 등을 한눈에 확인할 수 있습니다.

```
→  my-project git:(main) [Claude Sonnet 4.6] ctx:12%
```

> Status Line은 로컬에서 실행되며 API 토큰을 소비하지 않습니다.

---

## 설정 방법

### /statusline 명령어 (추천)

가장 간단한 방법입니다. 원하는 내용을 자연어로 설명하면 Claude가 스크립트를 생성하고 설정까지 자동으로 합니다.

```
/statusline 사용 모델, 컨텍스트 사용량, 사용량을 표현해주세요
```

인자 없이 실행하면 현재 쉘 프롬프트(PS1)를 기반으로 자동 구성합니다.

```
/statusline
```

### settings.json 수동 설정

`~/.claude/settings.json`에 직접 작성할 수도 있습니다.

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

인라인 명령어도 가능합니다:

```json
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] ctx:\\(.context_window.used_percentage // 0)%\"'"
  }
}
```

### 삭제

```
/statusline clear
```

또는 settings.json에서 `statusLine` 항목을 삭제합니다.

---

## 표시할 수 있는 데이터

Claude Code가 스크립트에 JSON으로 전달하는 주요 필드입니다.

### 모델 및 세션

| 필드 | 설명 |
|------|------|
| `model.display_name` | 현재 모델명 (예: "Sonnet 4.6") |
| `model.id` | 모델 ID (예: "claude-sonnet-4-6") |
| `session_name` | 세션 이름 (`/rename`으로 지정한 경우) |
| `version` | Claude Code 버전 |

### 컨텍스트 윈도우

| 필드 | 설명 |
|------|------|
| `context_window.used_percentage` | 컨텍스트 사용률 (%) |
| `context_window.remaining_percentage` | 남은 컨텍스트 (%) |
| `context_window.context_window_size` | 최대 컨텍스트 크기 (기본 200K, 1M 모델은 1M) |
| `context_window.total_input_tokens` | 누적 입력 토큰 |
| `context_window.total_output_tokens` | 누적 출력 토큰 |

### 비용 및 시간

| 필드 | 설명 |
|------|------|
| `cost.total_cost_usd` | 예상 비용 (USD) |
| `cost.total_duration_ms` | 세션 총 경과 시간 |
| `cost.total_lines_added` | 추가된 코드 줄 수 |
| `cost.total_lines_removed` | 삭제된 코드 줄 수 |

### 작업 디렉토리 및 Git

| 필드 | 설명 |
|------|------|
| `workspace.current_dir` | 현재 작업 디렉토리 |
| `workspace.project_dir` | Claude Code를 시작한 디렉토리 |
| `workspace.git_worktree` | git worktree 이름 (해당 시) |

### 속도 제한

| 필드 | 설명 |
|------|------|
| `rate_limits.five_hour.used_percentage` | 5시간 속도 제한 사용률 (0~100) |
| `rate_limits.seven_day.used_percentage` | 7일 속도 제한 사용률 (0~100) |

---

## 실전 예시

### 기본: 모델 + 컨텍스트

```
/statusline 모델명과 컨텍스트 사용률을 보여줘
```

결과: `[Claude Sonnet 4.6] ctx:42%`

### 추천: 모델 + git + 컨텍스트 + 비용

```
/statusline 모델명, git 브랜치, 컨텍스트 사용량, 비용을 보여줘
```

결과: `[Sonnet 4.6] main ctx:28% $0.05`

### 컨텍스트 프로그레스 바

```
/statusline 컨텍스트 사용량을 프로그레스 바로 표현해줘
```

결과: `[████░░░░░░] 42% ctx`

### 멀티라인

```
/statusline 첫 줄에 모델과 git 브랜치, 둘째 줄에 컨텍스트 바와 비용을 보여줘
```

---

## 직접 스크립트 작성하기

`/statusline`이 생성하는 스크립트를 직접 만들 수도 있습니다.

**1단계: 스크립트 작성**

`~/.claude/statusline.sh`:

```bash
#!/bin/bash
input=$(cat)

MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)

echo "[$MODEL] ${DIR##*/} | ctx:${PCT}%"
```

**2단계: 실행 권한 부여**

```bash
chmod +x ~/.claude/statusline.sh
```

**3단계: settings.json에 등록**

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

> `jq`가 필요합니다. macOS: `brew install jq`, Linux: `apt install jq`

### ANSI 색상 사용

컨텍스트 사용률에 따라 색상을 바꿀 수 있습니다:

```bash
#!/bin/bash
input=$(cat)
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)

if [ "$PCT" -lt 50 ]; then
  COLOR="\033[32m"  # 초록
elif [ "$PCT" -lt 80 ]; then
  COLOR="\033[33m"  # 노랑
else
  COLOR="\033[31m"  # 빨강
fi

echo -e "${COLOR}ctx:${PCT}%\033[0m"
```

---

## 추가 옵션

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| `padding` | 좌우 여백 (문자 수) | `0` |
| `refreshInterval` | 주기적 새로고침 간격 (초). 시계나 백그라운드 작업 상태 표시 시 유용 | 이벤트 기반만 |

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2,
    "refreshInterval": 5
  }
}
```

### 업데이트 시점

- 새로운 Claude 응답 후
- 권한 모드 변경 시
- vim 모드 토글 시
- `refreshInterval` 설정 시 해당 주기마다

> 300ms 디바운싱이 적용되어 빠른 연속 변경은 한 번만 업데이트됩니다.

---

## 왜 설정해야 할까

컨텍스트 사용량을 항상 볼 수 있으면 [컨텍스트 최적화]({{< relref "context-optimization" >}})를 훨씬 효과적으로 할 수 있습니다.

- `ctx:30%` → 여유 있음, 계속 작업
- `ctx:60%` → `/compact` 고려
- `ctx:80%` → 지금 바로 `/compact` 또는 `/clear`
- `ctx:90%+` → Auto-Compact 발동 직전. 성능이 이미 떨어지고 있을 수 있음

비용(`$0.05`)이나 속도 제한 사용률도 함께 표시하면, 모델을 전환하거나 작업 방식을 조절하는 타이밍을 놓치지 않습니다.
