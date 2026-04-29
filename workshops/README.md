# 워크샵

핸즈온 단계별 가이드 모음. 카테고리별로 폴더를 나눠 관리한다.

## AI 에이전트 (`agents/`)

CLI / IDE 기반 AI 코딩·작업 에이전트.

- [Claude (기본)](./agents/claude-workshop.md) — Anthropic Claude 앱: 채팅 · 프로젝트 · 루틴 · Cowork / Scheduled / Dispatch
- [Claude Code](./agents/claude-code-workshop.md) — Anthropic 터미널/IDE 코딩 에이전트
- [Codex](./agents/codex-workshop.md) — OpenAI Codex CLI / IDE
- [Gemini CLI](./agents/gemini-cli-workshop.md) — Google Gemini CLI
- [마켓플레이스 비교](./agents/marketplaces.md) — Claude Code · Codex · Gemini 마켓플레이스 정리 (2026 기준)

## 코드 에디터 (`editors/`)

- [VSCode](./editors/vscode-workshop.md) — 설치 · UI · Source Control · 추천 확장

## 협업 / 형상관리 (`collaboration/`)

- [GitHub](./collaboration/github-workshop.md) — Git 설치 · SSH · 클론 · 커밋 · 푸시

## 노트 / PKM (`notes/`)

- [Obsidian](./notes/obsidian-workshop.md) — Vault · 문법 · 백링크 · 플러그인 · Git 연동

## 에이전트 확장 / 워크플로우 (`extensions/`)

에이전트 본체 위에 얹는 프레임워크 / 스킬팩의 핸즈온 가이드.

- [GSD (Get Shit Done)](./extensions/gsd-workshop.md) — Claude Code 위 스펙 드리븐 워크플로우. `/gsd-new-project`부터 ship까지 전체 사이클 + workspace/workstream
- [Superpowers](./extensions/superpowers-workshop.md) — Claude Code Skill 디시플린 (TDD/디버깅/검증). brainstorm → plan → execute 3단계 + 자동 발동 메타스킬
- [gstack](./extensions/gstack-workshop.md) — Garry Tan(YC) 스킬팩 30+ 개. `/office-hours`부터 `/land-and-deploy`까지 풀 사이클 + GBrain 영속 메모리
- [oh-my-opencode](./extensions/oh-my-opencode-workshop.md) — OpenCode 오케스트레이션 하니스. Sisyphus/Hephaestus/Oracle 등 6개 에이전트 협력 + Claude Code 호환 레이어
- [Hermes Agent](./extensions/hermes-workshop.md) — Nous Research, 자기 진화 학습 루프. CLI + 텔레그램/디스코드 채널 + Claude Code 위임 + 81개 빌트인 스킬
- [OpenClaw](./extensions/openclaw-workshop.md) 🦞 — 셀프호스트 멀티채널 비서. WhatsApp/Telegram/Slack/iMessage + Live Canvas + Wake word + 자율 워크플로우 + Claude Code 플러그인
- [Harness](./extensions/harness-workshop.md) — Claude Code "Team-Architecture Factory". 도메인 한 줄 → 6 패턴(Pipeline/Fan-out/Expert Pool/Producer-Reviewer/Supervisor/Hierarchical) 자동 선택 → 에이전트팀+스킬 자동 생성

## 활용 팁 (`tips/`)

- [Remote 작업 환경 구성](./tips/remote-workshop.md) — tmux + Tailscale + Termius + ntfy로 어디서든 에이전트 접속 & 알림

## 새 워크샵 추가 가이드

1. 카테고리 결정 (위 4개 폴더 중 하나, 아니면 새 폴더 생성 사유 명시)
2. 파일명: `<도구>-workshop.md` (마켓플레이스/심화 비교 같은 횡단 문서는 `marketplaces.md` 같이 자유)
3. 도구 본체 레퍼런스는 [`../docs/`](../docs/) 에 있음. 워크샵은 단계별 따라하기, docs는 도구 카드 형태로 분리
