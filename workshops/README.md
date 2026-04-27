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

## 새 워크샵 추가 가이드

1. 카테고리 결정 (위 4개 폴더 중 하나, 아니면 새 폴더 생성 사유 명시)
2. 파일명: `<도구>-workshop.md` (마켓플레이스/심화 비교 같은 횡단 문서는 `marketplaces.md` 같이 자유)
3. 도구 본체 레퍼런스는 [`../docs/`](../docs/) 에 있음. 워크샵은 단계별 따라하기, docs는 도구 카드 형태로 분리
