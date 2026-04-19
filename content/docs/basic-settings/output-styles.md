---
title: 출력 스타일 (Output Styles)
weight: 3.7
sources:
  - title: "Claude Code 공식 문서 - Output Styles"
    url: "https://code.claude.com/docs/ko/output-styles"
---

출력 스타일은 Claude Code의 **응답 방식**을 바꿉니다. 코딩 도구가 아닌 학습 도우미, 기술 문서 작성자, 또는 완전히 다른 용도의 에이전트로 만들 수 있습니다.

---

## 기본 제공 스타일

| 스타일 | 설명 | 추천 상황 |
|--------|------|-----------|
| **Default** | 소프트웨어 엔지니어링 작업에 최적화된 기본 스타일 | 일반적인 코딩 작업 |
| **Explanatory** | 작업을 완료하면서 "Insights"로 구현 이유와 패턴을 설명 | 코드를 이해하면서 작업하고 싶을 때 |
| **Learning** | Insights + 사용자가 직접 코드를 작성하도록 `TODO(human)` 마커를 남김 | 코딩을 배우면서 실습할 때 |

---

## 스타일 변경 방법

### /config에서 선택

```
/config → Output style → 원하는 스타일 선택
```

### settings.json에서 직접 설정

```json
{
  "outputStyle": "Explanatory"
}
```

> 출력 스타일은 세션 시작 시 적용됩니다. 변경 후 새 세션을 시작해야 반영됩니다.

---

## 커스텀 출력 스타일 만들기

Markdown 파일 하나로 나만의 출력 스타일을 만들 수 있습니다.

### 저장 위치

| 위치 | 적용 범위 |
|------|---------|
| `~/.claude/output-styles/` | 모든 프로젝트 |
| `.claude/output-styles/` | 현재 프로젝트만 |

### 파일 구조

```markdown
---
name: 기술 문서 작성자
description: 코드를 분석하고 기술 문서를 작성하는 스타일
keep-coding-instructions: false
---

# 기술 문서 작성 지침

당신은 기술 문서 작성 전문가입니다.

## 응답 규칙

- 모든 응답은 한국어로 작성합니다
- 코드 블록에는 반드시 언어를 명시합니다
- 복잡한 개념은 비유를 사용해 설명합니다
- 각 섹션은 한 줄 요약으로 시작합니다
```

### Frontmatter 옵션

| 필드 | 설명 | 기본값 |
|------|------|--------|
| `name` | 스타일 이름 (`/config`에서 표시) | 파일 이름 |
| `description` | 스타일 설명 | 없음 |
| `keep-coding-instructions` | 기본 코딩 지침을 유지할지 여부 | `false` |

> `keep-coding-instructions: false`이면 Claude Code의 기본 코딩 관련 시스템 프롬프트(테스트 검증, 코드 스타일 등)가 제거됩니다. 코딩 작업에도 사용할 스타일이라면 `true`로 설정하세요.

---

## 활용 예시

### 한국어 코딩 도우미

```markdown
---
name: Korean Dev
description: 한국어로 응답하고 코드 설명을 덧붙이는 스타일
keep-coding-instructions: true
---

# 응답 지침

- 모든 응답과 커밋 메시지를 한국어로 작성합니다
- 코드 변경 시 변경 이유를 한 줄로 설명합니다
- 에러 메시지는 원문과 한국어 해석을 함께 제공합니다
```

### 코드 리뷰어

```markdown
---
name: Code Reviewer
description: 코드 리뷰에 특화된 스타일
keep-coding-instructions: true
---

# 코드 리뷰 지침

- 보안 취약점을 최우선으로 검사합니다
- 각 지적사항에 심각도(Critical/Warning/Info)를 붙입니다
- 개선 제안에는 반드시 수정된 코드 예시를 포함합니다
- 잘된 부분도 언급합니다
```

---

## 출력 스타일 vs 다른 기능

| 기능 | 역할 | 활성 조건 |
|------|------|----------|
| **출력 스타일** | 시스템 프롬프트를 교체. 응답 형식, 톤, 구조를 바꿈 | 선택하면 항상 활성 |
| **CLAUDE.md** | 시스템 프롬프트 뒤에 추가. 프로젝트 규칙 전달 | 매 세션 자동 로드 |
| **Skills** | 특정 작업용 프롬프트. `/skill-name`으로 호출 | 호출 시 또는 자동 감지 시 |
| **Agents** | 별도 컨텍스트에서 특정 작업 처리. 모델/도구 지정 가능 | 위임 시 |

> 일관된 응답 스타일 → **출력 스타일**, 재사용 가능한 워크플로우 → **Skills**
