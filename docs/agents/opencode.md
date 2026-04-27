# OpenCode

[sst](https://sst.dev/) 팀에서 만든 오픈소스 TUI 코딩 에이전트. **클라이언트/서버 분리 구조**가 특징이라, 헤드리스 모드로 자동화에 잘 맞습니다.

---

## 핵심 특징

- **Go + TypeScript** 하이브리드 (서버: Bun/TS, 클라이언트: Go TUI)
- **Provider-agnostic** — Claude/GPT/Gemini/로컬 등 자유롭게
- **Headless 서버 모드** — TUI 없이 API/CLI로만 호출 가능 (CI, 스크립트 친화)
- **세션 공유** — 서버 띄워놓고 여러 클라이언트가 같은 세션 보기 가능
- **MCP 지원**

---

## 설치

```bash
# 한 줄 인스톨
curl -fsSL https://opencode.ai/install | bash

# 또는 npm
npm install -g opencode-ai

opencode --version
```

---

## 사용

### TUI 모드

```bash
cd ~/your-project
opencode
```

### Headless / 일회성 실행

```bash
opencode run "이 프로젝트 README 요약해줘"
opencode run "tests 디렉토리 전체 통과시켜줘" --auto
```

CI에서 `opencode run`으로 PR 자동 리뷰 같은 것도 가능.

### 서버 모드

```bash
opencode serve --port 4096
# 다른 터미널/머신에서 client로 접속
```

---

## 설정

```
~/.config/opencode/
├── config.json
└── sessions/

<project>/.opencode/
└── config.json
```

`config.json` 예시:

```json
{
  "model": "anthropic/claude-sonnet-4-6",
  "providers": {
    "anthropic": { "apiKey": "$ANTHROPIC_API_KEY" },
    "openai":    { "apiKey": "$OPENAI_API_KEY" },
    "xai":       { "apiKey": "$XAI_API_KEY", "baseUrl": "https://api.x.ai/v1" }
  },
  "tools": {
    "bash": { "enabled": true },
    "edit": { "enabled": true }
  }
}
```

---

## Crush와의 비교

| 항목 | OpenCode | Crush |
|---|---|---|
| 아키텍처 | 서버 + 클라이언트 | 모놀리식 TUI |
| Headless | 강점 | 제한적 |
| TUI 미려함 | 좋음 | 매우 좋음 (Charm) |
| 자동화 | 매우 적합 | 보통 |

**선택 가이드**: CI / 봇 / 자동화에 박을 거면 OpenCode. 그냥 데일리 드라이버는 Crush 또는 Claude Code.

---

## oh-my-opencode

OpenCode 위에 얹는 설정/플러그인 모음. 별도 문서 참조: [`../extensions/oh-my-opencode.md`](../extensions/oh-my-opencode.md)

---

## 참고

- 공식: https://opencode.ai/
- GitHub: https://github.com/sst/opencode
