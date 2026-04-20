---
title: "프로젝트별 계정 분리하기"
weight: 10
---

프로젝트마다 다른 Claude 계정(개인/회사 등)을 사용하고 싶을 때 활용하는 방법입니다. `CLAUDE_CONFIG_DIR` 환경변수로 설정 디렉토리를 분리하면, 계정별로 완전히 독립된 환경을 만들 수 있습니다. 쉘 별칭을 등록하면 `claude-work`, `claude-personal`처럼 간단히 전환할 수 있습니다.

## 핵심 원리

Claude Code는 인증 토큰, 설정 등 모든 계정 정보를 `~/.claude` 디렉토리에 저장합니다.
`CLAUDE_CONFIG_DIR` 환경변수로 이 경로를 바꾸면 계정을 완전히 분리할 수 있습니다.

## 설정 방법

**1단계 — 계정별 디렉토리 생성**

```bash
mkdir -p ~/.claude-personal ~/.claude-work
```

**2단계 — `~/.zshrc`에 alias 추가**

```bash
# 개인 계정
alias claude-personal='CLAUDE_CONFIG_DIR=~/.claude-personal claude'

# 회사 계정
alias claude-work='CLAUDE_CONFIG_DIR=~/.claude-work claude'
```

**3단계 — 적용 후 각 계정으로 최초 로그인**

```bash
source ~/.zshrc

claude-personal   # 개인 계정으로 로그인
claude-work       # 회사 계정으로 로그인
```

최초 1회만 로그인하면, 이후에는 alias만 바꿔도 즉시 계정이 전환됩니다.

## 사용 예시

```bash
# 개인 프로젝트 작업
cd ~/projects/my-blog
claude-personal

# 회사 프로젝트 작업
cd ~/work/api-server
claude-work
```
