# 에이전트 확장 / 프레임워크

`agents/` 의 본체 위에 얹어서 행동/워크플로우를 강화하는 도구·스킬·프레임워크 모음입니다.

---

## 카테고리

### Claude Code 스킬 / 워크플로우 시스템

| 도구 | 한 줄 정리 |
|---|---|
| [Superpowers](./superpowers.md) | TDD / 디버깅 / 계획 등 **방법론 디시플린**을 스킬로 강제 |
| [gstack](./gstack.md) | Garry Tan(YC) 의 풀 사이클 셋업 — CEO/Designer/Eng/QA 23개 역할 |
| [GSD (Get Shit Done)](./gsd.md) | 스펙 드리븐 컨텍스트 관리. 12개 런타임 지원 |
| [Harness](./harness.md) | **메타 팩토리** — 도메인 받아 위 같은 스킬팩을 자동 생성 (L3) |

세 개를 같이 쓰는 [실용 가이드 (DEV.to)](https://dev.to/imaginex/a-claude-code-skills-stack-how-to-combine-superpowers-gstack-and-gsd-without-the-chaos-44b3) 참고.

### 자기 진화 / 영속 비서 런타임

| 도구 | 한 줄 정리 |
|---|---|
| [Hermes Agent](./hermes.md) | Nous Research, 작업하면서 스킬을 만들고 다듬는 **학습 루프** |
| [OpenClaw](./openclaw.md) | WhatsApp/Telegram/Slack 등으로 응답하는 **셀프호스트 비서** |

[The New Stack — OpenClaw vs Hermes 비교](https://thenewstack.io/persistent-ai-agents-compared/)

### 에이전트별 부가팩

| 도구 | 본체 |
|---|---|
| [oh-my-opencode](./oh-my-opencode.md) | OpenCode를 풀장착해주는 메가팩 |

---

## 어떻게 골라 얹나

대부분의 확장은 **특정 에이전트와 짝**입니다.

| 본체 | 잘 어울리는 확장 |
|---|---|
| Claude Code | Superpowers, gstack, GSD, OpenClaw 통합 플러그인 |
| OpenCode | oh-my-opencode, GSD (포팅판) |
| 어디서나 | GSD (12개 런타임), Hermes (모델 무관) |
| 메신저 비서 컨셉 | OpenClaw, Hermes |

---

## 직접 만들기

확장이 마땅치 않으면 직접 만드는 게 빠릅니다. Claude Code 기준:

- **Slash command** — `.claude/commands/<name>.md` 한 파일
- **Subagent** — `.claude/agents/<name>.md` 한 파일
- **Skill** — `.claude/skills/<name>/SKILL.md`
- **Hook** — `.claude/settings.json` 의 `hooks` 항목
- **MCP 서버** — 외부 API 붙이기 (Python/TS SDK 있음)

자세히는 [`../tips/`](../tips/) 폴더 참고.
