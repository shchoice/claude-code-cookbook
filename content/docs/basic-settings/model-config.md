---
title: 모델 설정 (Model Configuration)
weight: 3.5
sources:
  - title: "Claude Code 공식 문서 - Model Configuration"
    url: "https://code.claude.com/docs/en/model-config"
---

Claude Code에서 사용할 모델을 선택하고 사고 깊이를 조절하는 방법을 정리합니다. Sonnet(일상 코딩), Opus(복잡한 추론), Haiku(단순 작업), opusplan(계획은 Opus + 실행은 Sonnet) 등 별칭으로 모델을 지정할 수 있고, effort level로 사고 깊이를 5단계로 조절할 수 있습니다.

---

## 사용 가능한 모델

### 모델 별칭

정확한 버전 번호를 몰라도 별칭으로 모델을 지정할 수 있습니다.

| 별칭 | 설명 |
|------|------|
| `default` | 계정 유형에 맞는 기본 모델로 되돌림 |
| `sonnet` | 일상적인 코딩 작업에 적합 |
| `opus` | 복잡한 추론이 필요한 작업에 적합 |
| `haiku` | 빠르고 가벼운 단순 작업용 |
| `sonnet[1m]` | Sonnet + 100만 토큰 컨텍스트 |
| `opus[1m]` | Opus + 100만 토큰 컨텍스트 |
| `opusplan` | plan 모드에서는 Opus, 실행 모드에서는 Sonnet으로 자동 전환 |

### 플랜별 기본 모델

| 플랜 | 기본 모델 |
|------|-----------|
| Max, Team Premium | Opus 4.7 |
| Pro, Team Standard, Enterprise, API | Sonnet 4.6 |
| Bedrock, Vertex, Foundry | Sonnet 4.5 |

> Opus 사용량이 임계치를 넘으면 자동으로 Sonnet으로 대체될 수 있습니다.

---

## 모델 변경 방법

우선순위가 높은 순서대로:

**1. 대화 중 변경**

```
/model opus
```

또는 `/model`만 입력하면 선택 화면이 뜹니다.

**2. 실행 시 지정**

```bash
claude --model opus
```

**3. 환경변수**

```bash
export ANTHROPIC_MODEL=opus
```

**4. settings.json (영구 설정)**

```json
{
  "model": "opus"
}
```

---

## opusplan — 추천 모델 조합

`opusplan`은 **계획은 Opus로, 실행은 Sonnet으로** 자동 전환하는 모델 별칭입니다. Opus의 추론 능력과 Sonnet의 효율을 한번에 활용합니다.

### 동작 방식

| 모드 | 사용 모델 | 역할 |
|------|-----------|------|
| plan 모드 | Opus | 아키텍처 설계, 복잡한 의사결정, 마이그레이션 계획 |
| 실행 모드 | Sonnet | 코드 작성, 파일 수정, 테스트 실행 |

plan 모드에서 나가면 자동으로 Sonnet으로 전환됩니다. 수동으로 모델을 바꿀 필요가 없습니다.

### 사용 방법

```bash
# CLI로 시작
claude --model opusplan

# 대화 중 전환
/model opusplan
```

settings.json으로 영구 설정:

```json
{
  "model": "opusplan"
}
```

### 추천 워크플로우

**1단계: plan 모드에서 방향 잡기** (Opus)

```
Shift + Tab → plan 모드 진입
```

```
인증 시스템을 OAuth2로 마이그레이션하는 계획을 세워줘.
현재 세션 기반 인증 코드를 분석하고, 변경할 파일과 순서를 정리해.
```

Opus가 코드베이스를 분석하고 상세한 계획을 작성합니다. `Ctrl + G`로 플랜을 에디터에서 직접 수정할 수도 있습니다.

**2단계: 실행** (자동으로 Sonnet 전환)

플랜을 승인하면 Sonnet으로 자동 전환되어 구현을 시작합니다. 승인 시 실행 방식을 선택할 수 있습니다:

- auto 모드로 실행 — 확인 없이 자율 구현
- acceptEdits로 실행 — 파일 수정은 자동, 명령어는 확인
- 매번 확인하며 실행 — 한 단계씩 검토

### 언제 opusplan을 쓸까

| 상황 | 추천 모델 |
|------|-----------|
| 여러 파일을 건드리는 큰 기능 구현 | `opusplan` |
| 레거시 코드 리팩터링/마이그레이션 | `opusplan` |
| 아키텍처 결정이 필요한 설계 | `opusplan` |
| 단순 버그 수정, 변수 이름 변경 | `sonnet` |
| 한 줄 수정, 오타 교정 | `sonnet` 또는 `haiku` |
| 비용보다 정확도가 최우선 | `opus` |

> `opusplan`의 plan 모드 Opus는 표준 200K 컨텍스트를 사용합니다. 1M 컨텍스트 자동 확장은 `opus` 모델에만 적용되고 `opusplan`에는 적용되지 않습니다.

---

## Effort Level — 사고 깊이 조절

모델이 응답 전에 얼마나 깊이 생각할지를 조절합니다. 낮을수록 빠르고 저렴하지만, 복잡한 문제에서는 정확도가 떨어집니다.

### 사용 가능한 레벨

| 레벨 | 용도 |
|------|------|
| `low` | 단순 작업. 빠르지만 추론 약함 |
| `medium` | 비용 절약이 중요한 일반 작업 |
| `high` | 대부분의 코딩 작업에 적합 |
| `xhigh` | Opus 4.7 기본값. 복잡한 에이전트 작업에 최적 |
| `max` | 최대 추론. 현재 세션에만 적용 |

모델별 지원 레벨:

| 모델 | 레벨 |
|------|------|
| Opus 4.7 | low, medium, high, xhigh, max |
| Opus 4.6, Sonnet 4.6 | low, medium, high, max |

### 변경 방법

```
# 인터랙티브 슬라이더
/effort

# 직접 지정
/effort high

# 기본값으로 복원
/effort auto
```

`/model` 선택 화면에서 좌우 화살표로도 조절할 수 있습니다.

CLI로 시작할 때:

```bash
claude --effort max
```

### UltraThink

프롬프트에 **"ultrathink"**를 포함하면 해당 턴에서만 더 깊이 사고합니다. effort level을 바꾸지 않고 한 번만 깊게 생각하게 하고 싶을 때 유용합니다.

```
ultrathink. 이 인증 시스템의 보안 취약점을 분석해줘
```

> "think", "think hard" 같은 문구는 사고 토큰을 추가로 할당하지 않습니다. 반드시 **"ultrathink"**를 사용하세요.

---

## 확장 컨텍스트 (1M)

Opus 4.7, Opus 4.6, Sonnet 4.6은 **100만 토큰 컨텍스트**를 지원합니다. 긴 세션이나 큰 코드베이스에서 유용합니다.

### 플랜별 지원

| 플랜 | Opus 1M | Sonnet 1M |
|------|---------|-----------|
| Max, Team, Enterprise | 구독에 포함 | 추가 사용량 필요 |
| Pro | 추가 사용량 필요 | 추가 사용량 필요 |
| API | 전체 이용 가능 | 전체 이용 가능 |

### 사용 방법

```
/model opus[1m]
/model sonnet[1m]
```

> Max, Team, Enterprise 플랜에서는 Opus가 자동으로 1M 컨텍스트로 확장됩니다. 별도 설정 불필요.

1M 컨텍스트를 비활성화하려면:

```bash
export CLAUDE_CODE_DISABLE_1M_CONTEXT=1
```

---

## 모델 선택 가이드

| 하고 싶은 것 | 추천 설정 |
|-------------|-----------|
| 비용 효율적인 일상 코딩 | `sonnet` (기본) |
| 계획은 정확하게, 실행은 빠르게 | `opusplan` |
| 복잡한 아키텍처 설계 | `opus` |
| 서브에이전트의 단순 작업 | `haiku` |
| 간단한 질문에 빠르게 응답 | `sonnet` + `/effort low` |
| 한 턴만 깊이 분석 | 프롬프트에 "ultrathink" |
| 긴 세션, 큰 코드베이스 | `opus[1m]` 또는 `sonnet[1m]` |
