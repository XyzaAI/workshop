# OpenClaw

[openclaw/openclaw](https://github.com/openclaw/openclaw) — **"내 디바이스에서 도는 개인 AI 비서"**. WhatsApp / Telegram / Slack / Discord / iMessage 등 평소 쓰는 메신저로 응답하는 셀프호스트 에이전트.

> 사용자 메시지의 "open claw"가 이것입니다. 🦞 (로고가 가재)

Hermes Agent와 자주 비교됨 — 둘 다 **영속 메모리 + 멀티채널 비서** 컨셉.

---

## 핵심 컨셉

```
나 ──(메신저 / WhatsApp / Telegram / iMessage / ...)──→ OpenClaw 데몬
                                                           ↓
                                       Claude / GPT / Gemini / DeepSeek / 로컬
                                                           ↓
                                     영속 메모리 / 도구 / 스케줄 / 자동화
```

"항상 켜져 있는" 비서. 메일/일정/검색/코드 같은 작업을 메신저로 위임.

---

## 지원 채널

(거의 모든 메신저)

WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, BlueBubbles, IRC, Microsoft Teams, Matrix, Feishu, LINE, Mattermost, Nextcloud Talk, Nostr, Synology Chat, Tlon, Twitch, Zalo, WeChat, QQ, WebChat.

---

## 설치

요구사항: Node 24 권장 (22.14+ 가능).

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

설치 마법사가 다음을 셋업:
- 사용할 LLM 프로바이더
- 연결할 메신저 채널
- 백그라운드 데몬 등록

---

## 모델

OpenAI, Anthropic, Google, DeepSeek, OpenRouter, Ollama 등. 채널/태스크별로 다른 모델 라우팅도 가능.

---

## Claude Code와의 관계

OpenClaw 자체는 **독립 런타임**이지만, Claude Code를 잘 통합한 변형이 활발히 만들어지고 있음.

### 주요 통합 / 관련 프로젝트

| 프로젝트 | 설명 |
|---|---|
| [Enderfga/openclaw-claude-code](https://github.com/Enderfga/openclaw-claude-code) | OpenClaw 플러그인 — Claude Code CLI를 **프로그래머블 헤드리스 코딩 엔진**으로 변신. 도구, 에이전트 팀, 멀티 모델 프록시 포함. |
| [TechNickAI/openclaw-config](https://github.com/TechNickAI/openclaw-config) | Claude Code용 OpenClaw 설정 — 영속 메모리 / 11개 통합 스킬 / 4개 자동 워크플로우 |
| [yoloshii/ClawMem](https://github.com/yoloshii/ClawMem) | Claude Code / Hermes / OpenClaw 공통 온디바이스 메모리 레이어 (Hooks + MCP + 하이브리드 RAG) |
| [ComposioHQ/secure-openclaw](https://github.com/ComposioHQ/secure-openclaw) | 500+ 앱 통합 + 영속 메모리 + 스케줄 알림 가능한 보안 강화판 |
| [mergisi/awesome-openclaw-agents](https://github.com/mergisi/awesome-openclaw-agents) | 19 카테고리 162 production-ready 에이전트 템플릿 |

---

## Hermes Agent와의 비교

[The New Stack 비교 기사](https://thenewstack.io/persistent-ai-agents-compared/).

| 항목 | OpenClaw | [Hermes Agent](./hermes.md) |
|---|---|---|
| 핵심 가치 | **셀프호스트 + 멀티채널 비서** | **자기 진화 학습 루프** |
| 기본 동작 | 메신저로 명령 → 작업 수행 | 사용 중 스킬 자동 생성/개선 |
| 메모리 | 영속 (설정 의존) | 영속 + 능동적 nudge |
| 채널 다양성 | 매우 많음 (20+) | 많음 (Telegram/Discord/Slack 등) |
| Claude Code 통합 | 활발 (별도 플러그인) | 활발 (CCC 포팅 등) |

**선택 가이드**:
- **메신저 기반 비서가 핵심** → OpenClaw
- **에이전트가 알아서 똑똑해지길** 원함 → Hermes
- 둘 다 깔아서 ClawMem 같은 공유 메모리로 묶는 것도 가능

---

## 라이선스

MIT.

---

## 참고

- 메인: https://github.com/openclaw/openclaw
- 공식 사이트: https://openclaw.ai/
- 호스팅 플랫폼 OneClaw: https://www.oneclaw.net/
- The New Stack 비교 기사: https://thenewstack.io/persistent-ai-agents-compared/
