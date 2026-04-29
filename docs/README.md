# 문서 모음

워크샵 전반에서 공통으로 참조하는 문서와 AI 도구/생산성 자료를 모은 폴더입니다.

각 워크샵 문서는 필요할 때 여기로 링크합니다.

---

## 공통 설치 가이드

기본 환경을 셋업할 때 먼저 보는 문서들입니다.

- [Homebrew 설치 가이드 (macOS)](./brew.md)
- [Node.js 설치 가이드 (macOS / Windows)](./nodejs.md)

---

## AI 코딩 에이전트 (`agents/`)

CLI / IDE 기반 AI 코딩 에이전트 본체를 정리합니다.

- [Claude Code](./agents/claude-code.md) — Anthropic, 터미널 + IDE
- [Gemini CLI](./agents/gemini-cli.md) — Google
- [Codex CLI](./agents/codex.md) — OpenAI
- [Grok Code](./agents/grok-code.md) — xAI
- [Aider](./agents/aider.md) — 오픈소스, 가장 오래된 코딩 에이전트 중 하나
- [Crush](./agents/crush.md) — Charmbracelet, TUI
- [OpenCode](./agents/opencode.md) — sst, 오픈소스 TUI 에이전트

전체 개요: [`agents/README.md`](./agents/README.md)

---

## 에이전트 확장 / 프레임워크 (`extensions/`)

위 에이전트들 위에 얹어 쓰는 스킬·플러그인·프레임워크 모음입니다.

- [Superpowers](./extensions/superpowers.md) — Claude Code Skill 기반 동작 디시플린 (TDD/디버깅 등)
- [gstack](./extensions/gstack.md) — Garry Tan(YC CEO)의 Claude Code 풀-사이클 셋업, 23개 역할 스킬
- [GSD](./extensions/gsd.md) — Get Shit Done, 스펙 드리븐 컨텍스트 관리 (12개 런타임)
- [Hermes Agent](./extensions/hermes.md) — Nous Research, 자기 진화 학습 루프
- [OpenClaw](./extensions/openclaw.md) — 셀프호스트 메신저 비서 (WhatsApp/Telegram 등)
- [oh-my-opencode](./extensions/oh-my-opencode.md) — OpenCode "battery included" 메가팩
- [Harness](./extensions/harness.md) — Claude Code 메타 팩토리. 도메인 → 6 패턴 → 에이전트팀+스킬 자동 생성

전체 개요: [`extensions/README.md`](./extensions/README.md)

---

## 터미널 / 멀티플렉서 (`terminal/`)

AI 에이전트를 굴릴 때 필수에 가까운 터미널 환경입니다.

- [tmux](./terminal/tmux.md) — 가장 표준적인 터미널 멀티플렉서
- [cmux](./terminal/cmux.md) — Claude/AI 친화적 멀티플렉서

전체 개요: [`terminal/README.md`](./terminal/README.md)

---

## 활용 팁 (`tips/`)

도구 단위가 아니라 "이렇게 쓰면 좋다"는 노하우 모음입니다.

- [알림 / Notifications](./tips/notifications.md) — 작업 끝났을 때 효율적으로 받기
- [Hooks](./tips/hooks.md) — 자동화 트리거 만들기
- [MCP Setup](./tips/mcp-setup.md) — Model Context Protocol 서버 연결
- [생산성 팁](./tips/productivity.md) — 일반 패턴/단축키/세션 관리
- [Remote 작업 환경](./tips/remote.md) — 서버 + SSH + 모바일로 어디서든 에이전트 접속

전체 개요: [`tips/README.md`](./tips/README.md)

---

## 워크샵 문서

핸즈온 형태의 단계별 가이드는 [`../workshops/`](../workshops/) 에 있습니다.
