# AI 코딩 에이전트

터미널/IDE 환경에서 동작하는 AI 코딩 에이전트 본체들을 정리한 폴더입니다.

여기 있는 도구들은 "에이전트 본체" 그 자체이고, 이들 위에 얹는 스킬/플러그인은 [`../extensions/`](../extensions/) 에 정리합니다.

---

## 한눈에 비교

| 도구 | 제공자 | 모델 | 위치 | 라이선스 | 비고 |
|---|---|---|---|---|---|
| [Claude Code](./claude-code.md) | Anthropic | Claude Opus/Sonnet/Haiku | CLI + VSCode/JetBrains/Web | 상용 (Pro 이상 또는 API) | 가장 보편적, MCP/Hooks/Skills 지원 |
| [Gemini CLI](./gemini-cli.md) | Google | Gemini 2.5 Pro/Flash | CLI | 무료 + 유료 | Google AI Studio 키 무료 |
| [Codex CLI](./codex.md) | OpenAI | GPT-5 / o-시리즈 | CLI + Web/IDE | ChatGPT Plus 이상 | ChatGPT 계정과 연결 |
| [Grok Code](./grok-code.md) | xAI | grok-code-fast | CLI + IDE | API | 빠르고 저렴, 컨텍스트 큰 편 |
| [Aider](./aider.md) | 오픈소스 | 다중 (Claude/GPT/Gemini 등) | CLI | Apache 2.0 | git 친화적, 가장 성숙 |
| [Crush](./crush.md) | Charmbracelet | 다중 | TUI | 오픈소스 | 매우 예쁜 TUI, 다중 모델 |
| [OpenCode](./opencode.md) | sst | 다중 | TUI | 오픈소스 (MIT) | 클라이언트/서버 분리 구조 |

---

## 어떤 걸 골라야 하나

**"하나만 익혀야 한다면"** → Claude Code. 가장 안정적이고 생태계(MCP, Skills, Hooks)가 잘 잡혀 있음.

**"공짜로 시작"** → Gemini CLI (Google AI Studio 키 무료) 또는 Aider + 무료 모델.

**"OSS 선호 / 비공개 데이터"** → OpenCode 또는 Aider + 로컬 모델(Ollama 등).

**"여러 모델 한 번에 비교"** → Crush 또는 OpenCode. 모델 스위칭이 쉬움.

**"속도 우선"** → Grok Code (grok-code-fast 모델이 빠름).

---

## 공통적인 패턴

에이전트마다 명령어는 다르지만 큰 흐름은 비슷합니다.

1. **설치** — 보통 `npm i -g <pkg>` 또는 `brew install`
2. **인증** — 계정 로그인 또는 API 키
3. **프로젝트 폴더에서 실행** — 폴더가 컨텍스트가 됨
4. **자연어로 지시** — 파일 읽기/수정/셸 실행
5. **변경 검토** — diff 확인 후 승인
6. **커밋/푸시** — 보통 에이전트가 직접 수행

각 도구별 자세한 셋업은 개별 문서 참조.

---

## 비용/요금

- **상용 (Claude Code, Codex)** — 구독료 또는 API 사용량
- **API 종량제 (Grok Code 등)** — 토큰당 과금
- **무료 가능** — Gemini CLI (Google AI Studio 키), OpenCode/Aider + 로컬 모델

대부분의 에이전트는 **여러 모델 백엔드**를 지원하므로 (특히 OSS 계열) 비용/품질 트레이드오프를 직접 조절할 수 있습니다.
