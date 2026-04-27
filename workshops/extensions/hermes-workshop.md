# Hermes Agent 핸즈온 워크샵

[Hermes Agent](../../docs/extensions/hermes.md) (NousResearch/hermes-agent) — **자기 진화 학습 루프**가 핵심인 셀프호스트 비서. CLI 기반 + 메신저 채널 (Telegram/Discord/Slack/WhatsApp 등) + **81+ 빌트인 스킬**.

이 워크샵에서는:
- 한 줄 인스톨로 Hermes 띄우기
- 모델 선택 (Anthropic / OpenRouter / Ollama 등)
- 첫 대화 + **터미널 도구** 사용 시연
- 텔레그램 채널 연결로 모바일에서 Hermes 부르기
- **Claude Code 위임** — Hermes에서 코딩 작업을 Claude Code 에 떠넘기기
- 자기 학습 — 같은 패턴 반복 시 자동으로 스킬 생성되는 모습
- `hermes doctor` 진단 / 트러블슈팅

매 단계 ✅ 결과 확인 체크포인트가 있습니다.

> 사전 학습: [Claude Code 워크샵](../agents/claude-code-workshop.md) (Claude Code 위임 부분에 필요)
> 레퍼런스: [`docs/extensions/hermes.md`](../../docs/extensions/hermes.md)

---

## 0. 사전 준비

| 항목 | 확인 |
|---|---|
| OS | Linux / macOS / WSL2 / Android (Termux) |
| 디스크 | 1GB+ 여유 |
| 모델 API 키 (1개+) | Anthropic / OpenAI / OpenRouter / Nous Portal 중 하나 |
| (선택) 텔레그램 봇 | 채널 연결 시 |
| (선택) Docker | sandboxed 실행 시 |

> ⚠️ **컨텍스트 64K+ 모델만 동작**. Haiku 4.5, GPT-3.5, 작은 로컬 모델은 부적합.

---

## 1. 한 줄 설치

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

설치 마지막에 "PATH 추가" 안내가 나옵니다. 셸 재시작:

```bash
exec $SHELL
hermes --version
```

### ✅ 결과 확인

```bash
ls ~/.hermes/
# config.yaml  .env  skills/  data/  ...

cat ~/.hermes/config.yaml
# 기본값들이 채워져 있어야 함
```

설치 디렉토리: `~/.hermes/`
- `config.yaml` — 비밀 아닌 설정
- `.env` — API 키 등 비밀
- `skills/` — 빌트인 + 자동 생성 스킬
- `data/` — 영속 메모리 / 세션

---

## 2. 모델 선택

```bash
hermes model
```

인터랙티브 선택지가 뜹니다:

```
Select your AI provider:
  1. Nous Portal       (Hermes-3, 무료 티어)
  2. OpenRouter        (다중 모델)
  3. Anthropic Claude  (Sonnet 4.6 / Opus 4.7 추천)
  4. OpenAI            (GPT-5)
  5. Ollama (local)    (Hermes-3-Llama 등)
  6. Custom endpoint
```

**처음이라면 Anthropic 또는 OpenRouter 추천**. Sonnet 4.6 정도면 충분.

API 키를 묻습니다:

```
Enter ANTHROPIC_API_KEY: sk-ant-...
```

`~/.hermes/.env` 에 저장됨.

### ✅ 결과 확인

```bash
hermes doctor
```

다음 같은 출력이 나오면 OK:

```
✅ Installation: ~/.hermes/
✅ Config: config.yaml found
✅ Provider: anthropic (claude-sonnet-4-6)
✅ Context window: 200K tokens
✅ Skills: 81 built-in
✅ Memory: ready
⚠️  Channels: none configured (CLI only)
```

문제 있으면 같은 명령이 권장 fix 도 알려줌.

---

## 3. 첫 대화

```bash
hermes
```

또는 모던 TUI:

```bash
hermes --tui
```

```
Hermes > 안녕? 오늘 날짜 알려줘.
```

Hermes 가 **터미널 도구**를 가지고 있으므로 `date` 같은 명령을 알아서 실행:

```
[tool: bash]
$ date
Sun Apr 27 16:30:00 KST 2026

오늘은 2026년 4월 27일 일요일이에요.
```

### 슬래시 커맨드

`/` 만 치면 자동 완성으로 가능한 명령 목록.

| 명령 | 동작 |
|---|---|
| `/help` | 도움말 |
| `/skills` | 사용 가능 스킬 목록 |
| `/memory` | 영속 메모리 보기 / 편집 |
| `/model` | 모델 변경 |
| `/clear` | 대화 초기화 |
| `/save` | 현재 컨텍스트 메모리에 저장 |
| `/channels` | 메신저 채널 관리 |
| `/quit` | 종료 |

### 키 입력

| 키 | 동작 |
|---|---|
| `Alt + Enter` | 멀티라인 입력 |
| `Ctrl + C` | 작업 중단 (종료 아님) |
| `Ctrl + D` | 종료 |

### 다음 세션 이어서

```bash
hermes --continue
```

마지막 세션 컨텍스트 그대로 복원. 메모리는 항상 유지됨.

### ✅ 결과 확인

종료 (`/quit`) 후 다시:

```bash
hermes --continue
> 방금 알려준 오늘 날짜 다시 한 번 말해줘
```

기억하고 답하면 OK.

---

## 4. 빌트인 스킬 둘러보기

```
> /skills
```

81+ 개 스킬이 카테고리별로:

| 카테고리 | 예시 |
|---|---|
| **AI 코딩 위임** | claude-code, openai-codex, opencode |
| **Apple 통합** | notes, reminders, findmy, imessage |
| **개발** | github-pr, code-review, gh-workflow |
| **생산성** | airtable, google-workspace, linear, notion, pdf-tools |
| **미디어** | spotify, youtube, gif-search |
| **창작** | ascii-art, diagrams, pixel-art, design |
| **ML/Data** | fine-tuning, llm-eval, model-serving |
| **리서치** | arxiv, polymarket, rss-feeds |

### 스킬 동작 보기

```
> 최근 ML 논문 5개 (트랜스포머 관련) 찾아서 abstract 정리해줘
```

`arxiv` 스킬이 자동 발동:

```
[skill: arxiv]
Searching: "transformer 2026"
[5 papers found]
```

### ✅ 결과 확인

```bash
ls ~/.hermes/skills/
# arxiv/  github/  notion/  ... (81 directories)
# + 자동 생성된 사용자 스킬도 시간 지나면 추가됨
```

---

## 5. 영속 메모리 — 핵심 차별 기능

Hermes 의 가장 큰 강점. 다음 같은 패턴이 가능합니다:

```
> 내 동료 김철수는 PostgreSQL 전문가야. 그 친구한테 SQL 질문 하면 좋다는 거 기억해.
```

```
[memory: noted]
"동료 김철수 — PostgreSQL 전문가. SQL 질문 시 추천."
```

세션 종료 후 **다른 날** 다시 켜면:

```bash
hermes
> SQL 인덱스 잘 만드는 거 누구한테 물어볼까?
```

```
김철수 씨한테 물어보세요. PostgreSQL 전문가시라고 알려주셨어요.
```

### 메모리 직접 관리

```
> /memory
```

저장된 메모리 보기 / 검색 / 삭제.

수동 추가:

```
> /memory add 우리 팀 코드 컨벤션: tabs는 안 쓰고 4칸 스페이스
```

### 자동 nudge

Hermes 는 가끔 스스로 "이거 메모리에 넣을 가치 있어 보임" 이라며 제안:

```
[memory: nudge]
"방금 한 시간 동안 같은 디버깅 패턴이 반복됐어요.
 이걸 패턴으로 박제할까요?"
```

승인하면 → 다음 세션부터 같은 상황에 자동 발동 가능.

### ✅ 결과 확인

```bash
ls ~/.hermes/data/memory/
# 카테고리별 마크다운 파일들

cat ~/.hermes/data/memory/colleagues.md
# 김철수 — PostgreSQL 전문가, ...
```

---

## 6. 자기 진화 — 스킬 자동 생성

복잡한 작업을 처리한 뒤, Hermes 가 **그 절차를 스킬로 박제**합니다.

### 시연

```
> 이 GitHub 이슈 ( https://github.com/<repo>/issues/123 ) 읽고 요약해서
  Notion 데이터베이스에 카드로 추가해줘. 우선순위는 라벨 보고 자동 분류.
```

Hermes 가 `github-issue` + `notion` 스킬을 조합해 처리. 끝나면:

```
[skill: nudge]
"같은 패턴 — github 이슈 → notion 카드 — 을 반복하시는 것 같습니다.
 이 절차를 스킬로 만들까요? 다음에 'gh-to-notion <url>' 한 줄로 호출 가능."
```

승인:

```
> 응 만들어줘
```

### ✅ 결과 확인

```bash
ls ~/.hermes/skills/gh-to-notion/
# SKILL.md  prompt.md  metadata.json

cat ~/.hermes/skills/gh-to-notion/SKILL.md
```

다음 세션부터:

```
> /gh-to-notion https://github.com/<repo>/issues/124
```

→ 학습된 절차 그대로 실행. **사용할수록 더 다듬어짐**.

---

## 7. 텔레그램 채널 연결 — "어디에서나 Hermes"

### 봇 만들기

1. 텔레그램에서 [@BotFather](https://t.me/BotFather) 와 대화
2. `/newbot` → 이름 / username 입력
3. 토큰 받음 (`123456:ABC...`)

### Hermes 에 등록

```
> /channels add telegram
```

봇 토큰 입력 → Hermes 가 알아서 webhook 또는 polling 셋업.

### ✅ 결과 확인

봇과 대화 시작:

```
[Telegram → 내 봇]
"오늘 할 일 정리해줘"
```

Hermes 가 응답. **CLI 와 같은 인격** — 메모리 / 스킬 / 컨텍스트 공유.

```bash
cat ~/.hermes/config.yaml | grep -A 5 channels
# channels:
#   telegram:
#     enabled: true
#     bot_username: my_hermes_bot
```

### Discord / Slack / WhatsApp 도 동일

```
> /channels add discord
> /channels add slack
> /channels add whatsapp
```

각 플랫폼에서 토큰/앱 만든 뒤 등록.

### 채널 정책

```
> /channels policy
```

- 어떤 채널이 어떤 스킬 사용 가능
- 응답 길이 제한
- 권한 (예: WhatsApp은 read-only)

---

## 8. Claude Code 위임 — 코딩은 Claude 한테

Hermes 의 강점은 비서/통합/메모리지만 **순수 코딩**은 Claude Code 가 강합니다. 위임 패턴:

```
> 우리 my-todo-app 프로젝트 ( ~/Desktop/my-todo-app ) 의 build 명령이
  실패한대. Claude Code 띄워서 고치게 해줘. 다 끝나면 PR 만들고 알려줘.
```

Hermes 가 `claude-code` 스킬로 위임:

```
[skill: claude-code]
$ cd ~/Desktop/my-todo-app
$ claude --headless "build 명령 실패하는 거 디버그하고 고쳐줘. 끝나면 PR 만들어"

[Claude Code working...]
[2분 후]
[Claude Code done — PR #45 created: https://github.com/.../pull/45]

📬 [Telegram 알림 — 텔레그램 채널 켜져 있다면]
"PR #45 가 만들어졌어요. 리뷰 부탁드려요."
```

### ✅ 결과 확인

```bash
gh pr view 45 --web
# Claude Code 가 만든 PR

# 텔레그램 알림 도착했는지
```

> 💡 같은 위임 패턴: `openai-codex`, `opencode` 스킬도 동일 방식.

---

## 9. Sandboxed 실행 — 보안 강화

기본은 호스트 머신에서 직접 실행. 위험한 명령이 걱정되면 sandbox:

### 옵션 1: Docker

```bash
hermes config terminal.backend docker
```

이후 모든 셸 명령이 격리된 컨테이너에서 실행됨.

### 옵션 2: SSH (원격 서버)

```bash
hermes config terminal.backend ssh \
  --host my-server.com --user kang
```

### 옵션 3: Daytona / Modal (관리형 sandbox)

```bash
hermes config terminal.backend daytona
hermes config terminal.backend modal
```

### ✅ 결과 확인

```
> 'whoami' 실행해줘
```

응답:

```
[tool: bash, backend: docker]
hermes-sandbox
```

호스트 사용자가 아니라 컨테이너 내부 사용자가 나오면 격리 동작.

---

## 10. 음성 모드

```
> /voice on
```

마이크 입력 / TTS 출력. 쓸 만한 시나리오:
- 운전 중 스마트폰 → 텔레그램 음성 메시지 → Hermes 응답
- 데스크톱 작업 중 핸즈프리

### ✅ 결과 확인

```
> /voice
[Voice mode active]
```

마이크 권한 허용 → 말하면 받아쓰기.

---

## 11. MCP 서버 추가

Hermes 도 **MCP** 호환:

```
> /mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Documents
```

이후 ~/Documents 의 파일들이 Hermes의 도구로 노출.

자세히는 [`docs/tips/mcp-setup.md`](../../docs/tips/mcp-setup.md).

### ✅ 결과 확인

```
> /mcp list
```

활성 MCP 서버 출력.

---

## 12. 진단 / 트러블슈팅

### 만능 진단 명령

```bash
hermes doctor
```

상태 + 권장 fix.

### 자주 나는 이슈

#### 모델이 응답 없음

```
> /model
```

다른 모델로 전환 → 같은 증상이면 API 키 / 잔액 확인.

#### 컨텍스트 부족 에러

```
context window: 64K required, model has 32K
```

→ `hermes model` 다시 → 64K+ 모델 선택. 작은 모델 (Haiku, gpt-3.5) 은 부적합.

#### 채널 메시지 안 옴

```bash
hermes doctor --channels
```

webhook URL / token 검증.

#### 스킬이 자동 생성 안 됨

```bash
cat ~/.hermes/config.yaml | grep skill_creation
# enabled: true
```

비활성이면 켜기:

```bash
hermes config skill_creation.enabled true
```

### 복구 시퀀스

망가진 상태:

```bash
hermes doctor --recover
# 또는
hermes init --reset
```

`~/.hermes/data/` 백업 후 처음부터.

---

## 13. 전체 흐름 요약

```
[설치]
  curl install.sh | bash
  ↓
hermes model               → 모델 + API 키
  ↓
hermes doctor              → 환경 검증
  ↓
hermes (CLI)               → 첫 대화
  ↓
대화하며 정보 / 패턴 누적
  ↓
[영속 메모리]                같은 정보를 또 안 말해도 됨
[자동 스킬 생성]              반복 패턴이 스킬화 → 다음엔 한 줄로 호출
  ↓
[채널 추가]
  /channels add telegram   → 모바일 / 카톡 / 슬랙 어디서든
  ↓
[위임]
  코딩은 Claude Code 로
  리서치는 arxiv / context7
  검색은 exa / grep.app
  ↓
[Sandbox]                   필요 시 Docker / SSH / Modal
```

수고하셨습니다 🎉

---

## 14. 다른 스택과 함께

| 조합 | 효과 |
|---|---|
| Hermes + Claude Code (위임) | Hermes 가 비서, Claude Code가 코더 |
| Hermes + [OpenClaw](../../docs/extensions/openclaw.md) | 둘 다 메신저 비서 — 메모리 공유는 [ClawMem](https://github.com/yoloshii/ClawMem) |
| Hermes + [Superpowers](./superpowers-workshop.md) (Claude Code 측) | Hermes 위임받은 Claude Code 가 디시플린대로 동작 |

---

## 15. 더 보기

- 레퍼런스: [`docs/extensions/hermes.md`](../../docs/extensions/hermes.md)
- 메인: https://github.com/NousResearch/hermes-agent
- 공식 사이트: https://hermes-agent.nousresearch.com
- 스킬 허브: https://hermes-agent.nousresearch.com/docs/skills/
- Quickstart: https://hermes-agent.nousresearch.com/docs/getting-started/quickstart
- awesome 모음: https://github.com/0xNyk/awesome-hermes-agent
- Claude Code 통합 변형:
  - https://github.com/AlexAI-MCP/hermes-CCC (Hermes를 Claude Code 안으로)
  - https://github.com/sypsyp97/claude-hermes (lightweight)
- OpenClaw vs Hermes: https://thenewstack.io/persistent-ai-agents-compared/
