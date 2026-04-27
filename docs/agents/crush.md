# Crush

Charmbracelet에서 만든 **TUI 미려한** 오픈소스 코딩 에이전트. Glamour 렌더링 / Bubble Tea 기반이라 터미널 안에서의 UX가 가장 좋다는 평이 많습니다.

---

## 핵심 특징

- **Go로 작성** — 단일 바이너리 배포, 빠름
- **다중 LLM 백엔드** — Claude / GPT / Gemini / Grok / 로컬 모두 지원
- **세션 영속화** — 작업 컨텍스트가 디스크에 저장됨, 종료/재개 자유
- **MCP 지원** — Model Context Protocol 기본 내장
- **여러 세션 병렬** — 한 프로젝트에서 여러 작업을 동시에 굴릴 수 있음

---

## 설치

```bash
# Homebrew
brew install charmbracelet/tap/crush

# Go
go install github.com/charmbracelet/crush@latest

# npm
npm install -g @charmland/crush

crush --version
```

---

## 인증

환경변수 또는 `~/.config/crush/config.json` 에 provider별 키 등록.

```bash
export ANTHROPIC_API_KEY="..."
export OPENAI_API_KEY="..."
export GEMINI_API_KEY="..."
export XAI_API_KEY="..."
```

```bash
cd ~/your-project
crush
```

---

## 모델 스위칭

`Ctrl+P` (또는 슬래시 커맨드) 로 진행 중 세션의 모델을 갈아끼울 수 있음. 같은 작업의 같은 컨텍스트를 다른 모델이 이어받게 할 수 있어서 **모델 비교**에 유용.

---

## 설정

```
~/.config/crush/
├── config.json     # provider, 단축키, 테마
└── sessions/       # 세션 저장
```

`config.json` 예시:

```json
{
  "providers": {
    "anthropic": { "apiKey": "$ANTHROPIC_API_KEY", "default": "claude-sonnet-4-6" },
    "openai":    { "apiKey": "$OPENAI_API_KEY" },
    "xai":       { "apiKey": "$XAI_API_KEY", "baseUrl": "https://api.x.ai/v1" }
  },
  "mcp": [
    { "name": "filesystem", "command": ["npx","-y","@modelcontextprotocol/server-filesystem","~/Documents"] }
  ]
}
```

---

## 강점

- **눈에 즐거움** — 가장 완성도 높은 터미널 UI 중 하나
- **세션 관리** — 어제 하던 작업 그대로 이어서
- **공급사 종속 없음** — 한 도구로 여러 모델
- **빠른 시작** — Go 단일 바이너리

## 약점

- 생태계는 Claude Code/Aider 대비 작음
- Hooks 같은 자동화 훅이 적음 (개발 중)
- Skills/Subagents 같은 메타 추상화는 부족

---

## 비슷한 도구

- [OpenCode](./opencode.md) — sst, 클라이언트/서버 분리 구조
- [Claude Code](./claude-code.md) — 생태계 풍부
- [Aider](./aider.md) — git 친화적

---

## 참고

- GitHub: https://github.com/charmbracelet/crush
- Charmbracelet 다른 제품: glow, gum, lazygit (인접 생태계)
