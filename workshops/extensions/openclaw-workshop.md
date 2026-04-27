# OpenClaw 핸즈온 워크샵

[OpenClaw](../../docs/extensions/openclaw.md) (openclaw/openclaw) 🦞 — **자기 디바이스에서 도는 개인 AI 비서**. WhatsApp / Telegram / Slack / Discord / Signal / iMessage 등으로 응답. 50+ 외부 서비스 통합 (Gmail / GitHub / Spotify / Obsidian / Calendar 등).

이 워크샵에서는:
1. 한 줄 인스톨 + `openclaw onboard --install-daemon`
2. 게이트웨이 셋업 + 모델 프로바이더 (Claude / GPT / Gemini / 로컬)
3. 텔레그램 채널 연결로 모바일에서 OpenClaw 호출
4. **Skills 셋업** — Gmail 자동 분류, Calendar 일일 브리핑
5. **Live Canvas** — 비서가 시각화하는 작업 화면
6. 음성 모드 + Wake word
7. **Sandbox 모드** — 그룹 채널에서 안전하게 실행
8. **Claude Code 연결** — Enderfga/openclaw-claude-code 플러그인으로 코딩 위임
9. 자율 워크플로우 (스케줄 / 트리거 기반)

매 단계 ✅ 결과 확인 체크포인트가 있습니다.

> 사전 학습: [Claude Code 워크샵](../agents/claude-code-workshop.md) — Claude Code 위임 부분에 필요
> 레퍼런스: [`docs/extensions/openclaw.md`](../../docs/extensions/openclaw.md)

---

## 0. 사전 준비

| 항목 | 확인 / 설치 |
|---|---|
| OS | macOS / Linux / Windows |
| Node.js 24 (또는 22.14+) | `node --version` |
| 디스크 1GB+ | 메모리/세션 영속화 공간 |
| 모델 API 키 (1개+) | Anthropic / OpenAI / Gemini / OpenRouter / 로컬 |
| (선택) 텔레그램 봇 | 채널 연결 시 |
| (선택) Docker | sandbox 모드 시 |

---

## 1. 설치

### 한 줄 인스톨

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

또는 npm:

```bash
npm install -g openclaw@latest
```

확인:

```bash
openclaw --version
```

### Onboard

설치 직후 한 번만:

```bash
openclaw onboard --install-daemon
```

마법사가 다음을 묻습니다:

| 단계 | 물어보는 것 |
|---|---|
| Gateway | 게이트웨이 프로세스 (메시지 수신/전달) |
| Workspace | 작업 디렉토리 (`~/.openclaw/`) |
| Model Provider | Anthropic / OpenAI / Gemini / DeepSeek / OpenRouter / Ollama |
| API Key | provider 별 API 키 |
| Default Channels | 어떤 채널부터 켤지 (나중에 추가 가능) |
| Daemon | 백그라운드 데몬 등록 (자동 시작) |

### ✅ 결과 확인

```bash
ls ~/.openclaw/
# config.yaml  data/  skills/  sessions/  logs/  ...

cat ~/.openclaw/config.yaml | head -20
# gateway: ...
# workspace: ~/.openclaw
# model:
#   provider: anthropic
#   default: claude-sonnet-4-6
# channels: []
# skills: []
```

데몬 상태:

```bash
openclaw status
# ✅ daemon: running (pid 12345)
# ✅ gateway: ready
# ✅ workspace: ~/.openclaw
# ⚠️ channels: 0 connected (CLI only)
```

---

## 2. 첫 대화 (CLI)

```bash
openclaw chat
```

또는 짧게:

```bash
openclaw
```

```
OpenClaw 🦞 > 안녕? 오늘 날짜 알려줘.
```

OpenClaw 가 시스템 도구를 가지고 있어서:

```
[tool: shell]
$ date
Sun Apr 27 17:00:00 KST 2026

오늘은 2026년 4월 27일 일요일입니다.
```

### 슬래시 커맨드

```
/help         도움말
/skills       사용 가능 스킬
/skills add   스킬 설치
/channels     채널 관리
/memory       영속 메모리
/session      세션 정보 / 분기
/canvas       Live Canvas 열기
/voice        음성 모드 on/off
/sandbox      샌드박스 토글
/quit         종료
```

### ✅ 결과 확인

```
> /skills
```

기본 설치된 스킬 일부 (서비스 통합):
- gmail / gcal / gdrive
- github
- notion / obsidian
- spotify / youtube
- calendar
- ...

---

## 3. 텔레그램 채널 연결

### 봇 만들기

1. 텔레그램에서 [@BotFather](https://t.me/BotFather) → `/newbot`
2. 이름 / username 입력 → 토큰 받음

### OpenClaw 등록

```bash
openclaw channels add telegram
```

또는 CLI 안에서:

```
> /channels add telegram
```

봇 토큰 입력 → OpenClaw 가 webhook 또는 polling 셋업.

### ✅ 결과 확인

봇과 첫 대화:

```
[Telegram 모바일]
"오늘 일정 알려줘"
```

OpenClaw 가 Calendar 스킬로 응답.

```bash
cat ~/.openclaw/config.yaml | grep -A 5 channels
# channels:
#   - type: telegram
#     bot_username: my_openclaw_bot
#     enabled: true
```

### Discord / Slack / iMessage / WhatsApp / Signal

```
> /channels add discord
> /channels add slack
> /channels add imessage    # macOS 만, BlueBubbles 또는 직접
> /channels add whatsapp
> /channels add signal
```

각 플랫폼 별 가이드는 `openclaw channels help <type>`.

> 💡 모든 채널이 **같은 인격** — 메모리 / 스킬 / 컨텍스트 공유.

---

## 4. 스킬 시스템 — Gmail 자동화 예제

OpenClaw 의 진짜 가치는 **자동화 스킬** 들. 데일리 활용 예 시연.

### Gmail 인증

```
> /skills add gmail
```

OAuth 흐름 → 권한 부여 → token 영속화 (`~/.openclaw/data/auth/gmail.json`).

### 활용 예

```
> 오늘 안 읽은 이메일 중에 정말 답변 필요한 것만 골라서 한 줄 요약 줘.
  광고 / 뉴스레터 / GitHub 알림 같은 거는 빼고.
```

OpenClaw 가:
1. Gmail API 로 unread 가져오기
2. 카테고리 라벨 분석
3. LLM 으로 요약 + 우선순위
4. 답장 필요 여부 예측

```
[Inbox 분석 — 23건 중 5건이 답변 필요]

1. 김철수 (고객사) — 다음 주 미팅 시간 제안 받았음 → 회신 필요
2. accounts@stripe.com — 카드 등록 실패 → 결제 정보 갱신 필요
3. ...
```

### ✅ 결과 확인

```bash
ls ~/.openclaw/data/auth/
# gmail.json  github.json  ... (인증된 서비스만)

ls ~/.openclaw/skills/
# gmail/   github/   ...
```

---

## 5. Live Canvas — 시각 작업 공간

OpenClaw 의 차별 기능. **에이전트가 작업하는 모습을 시각적으로**.

```
> /canvas
```

브라우저에 인터랙티브 캔버스가 뜨고, 거기서 OpenClaw 가:
- 그래프 / 차트 그리기
- HTML 시안 보여주기
- 진행 중 작업의 상태 보드
- 사용자가 직접 캔버스에서 항목 드래그 → 에이전트에 명령

### 시연

```
> 다음 5개 항목으로 칸반 보드 만들어 줘. 각 항목은 우선순위 H/M/L 카드로.
  - 회의록 정리
  - PR 리뷰 3건
  - DB 마이그레이션 검토
  - 휴가 신청
  - 노트북 충전기 사오기
```

캔버스에 칸반이 자동 렌더링. 카드를 드래그 → 우선순위 변경 → OpenClaw 에 자동 반영.

### ✅ 결과 확인

캔버스 URL 이 출력됨 (보통 `http://localhost:48xx/canvas/...`). 브라우저에서 열어 확인.

---

## 6. 영속 메모리

다른 비서 에이전트들과 마찬가지로:

```
> 우리 회사 PM은 박지민이고 매주 화요일 10시에 1on1 미팅이 있어.
  매 일요일 저녁에 다음 주 1on1 의제 정리하라고 알려줘.
```

OpenClaw 가:
- 메모리에 저장
- **자율 워크플로우** 로 스케줄 등록 (일요일 저녁 트리거)

### ✅ 결과 확인

```bash
ls ~/.openclaw/data/memory/
# people.md  preferences.md  schedules.md  ...

cat ~/.openclaw/data/memory/people.md
# - 박지민 (PM, 화 10시 1on1)

cat ~/.openclaw/data/memory/schedules.md
# - 일요일 19:00: "1on1 의제 정리 알림" → workflow:weekly-1on1-prep
```

---

## 7. 자율 워크플로우 — 스케줄 / 트리거

### 일일 브리핑 스케줄

```
> 매일 아침 8시에 다음을 텔레그램으로 보내줘:
  1. 오늘 일정 (Calendar)
  2. 답변 필요한 이메일 요약 (Gmail)
  3. 어제 GitHub 활동 요약
```

```
✅ Workflow created: daily-briefing
   Trigger: cron 0 8 * * *
   Channel: telegram (default)
   Skills: gcal, gmail, github
```

### ✅ 결과 확인

```bash
openclaw workflows list
# 1. daily-briefing      cron 0 8 * * *
# 2. weekly-1on1-prep    cron 0 19 * * 0
```

다음 날 아침 텔레그램 확인.

### 트리거 기반 (이벤트)

```
> GitHub 우리 organization의 PR 이 새로 열리면 디스코드 #engineering 채널에 알려줘.
  내가 owner / reviewer 인 것만.
```

```
✅ Workflow created: pr-watch
   Trigger: github webhook (pulls.opened)
   Filter: owner=me OR reviewer_requested=me
   Channel: discord #engineering
```

---

## 8. 음성 모드 + Wake Word

### 활성

```
> /voice on
```

macOS / iOS:
- Wake word 활성 ("OpenClaw" 또는 사용자 정의)
- ElevenLabs TTS / 시스템 TTS fallback

Android:
- 연속 음성 인터랙션 모드

```
🎤 듣는 중... ("OpenClaw" 라고 부르면 활성)
```

```
[사용자] "OpenClaw, 오늘 할 일 알려줘"
[OpenClaw 음성으로 응답]
```

### ✅ 결과 확인

```bash
cat ~/.openclaw/config.yaml | grep -A 5 voice
# voice:
#   enabled: true
#   wake_word: openclaw
#   tts: elevenlabs
```

---

## 9. Sandbox 모드 — 그룹/채널에서 안전하게

기본은 호스트에서 직접 실행 (메인 세션). 그룹 채널 / 모르는 사용자가 명령을 보내면 위험.

### Non-main 세션은 sandbox

`~/.openclaw/config.yaml`:

```yaml
agents:
  defaults:
    sandbox:
      mode: "non-main"
      backend: docker
```

이제 메인 세션 (개인 채팅) 은 호스트, 그룹 채널 / Discord / Slack 같은 다중 사용자 환경은 Docker 컨테이너 안에서.

### ✅ 결과 확인

그룹 채널에서:

```
[Discord 그룹]
"whoami 실행해줘"
```

응답:

```
[backend: docker]
openclaw-sandbox
```

호스트 사용자명 대신 컨테이너 내부 사용자가 나오면 격리 동작.

---

## 10. Claude Code 통합 — Enderfga/openclaw-claude-code 플러그인

OpenClaw 자체로도 코딩 가능하지만, **Claude Code** 를 백엔드로 끼우면 훨씬 강력합니다.

### 설치

```bash
curl -fsSL https://raw.githubusercontent.com/Enderfga/openclaw-claude-code/main/install.sh | bash
# 또는
npm install -g @enderfga/openclaw-claude-code
```

> 요구사항: Node 22+ / Claude Code CLI 2.1+

### 무엇이 추가되나

- **Multi-Engine** — Claude Code, OpenAI Codex, Google Gemini, Cursor Agent, 커스텀 CLI 통합 인터페이스
- **Multi-Agent Council** — 여러 에이전트 병렬 + git worktree 격리 + consensus voting
- **27 Tools** — 세션 라이프사이클 / cross-session 메시지 (inbox) / 에이전트 팀
- **Persistent Sessions** — 7일 영속, 재시작에도 resume
- **OpenAI-호환 API** — ChatGPT-Next-Web / Open WebUI 같은 webchat 프론트엔드의 백엔드로 사용 가능

### Ultraplan / Ultrareview

```
> /ultraplan 결제 시스템에 PG 추가하기
```

전용 플래닝 세션이 프로젝트를 깊게 탐색해서 상세 구현 전략 도출.

```
> /ultrareview
```

5~20개 버그 헌터 에이전트가 코드베이스를 다양한 각도로 동시 리뷰.

### Cost Tracking + Model Switching

```
> /cost
```

실시간 토큰 / 비용 (모델별 가격 적용).

```
> /switch-model claude-opus-4-7
```

대화 중간에 모델 핫스왑.

### ✅ 결과 확인

```bash
openclaw-claude-code --version

# 27 tools 등록 확인
openclaw-claude-code tools list
```

---

## 11. Multi-Agent Council 시연

```
> /council 이 함수의 시간복잡도 분석하고 최적화 제안.
  3명에게 동시에 묻고 consensus 추출.
```

흐름:

```
[Council launches]
  Agent 1 (Claude Sonnet) — git worktree A
  Agent 2 (GPT-5)         — git worktree B
  Agent 3 (Gemini Pro)    — git worktree C
       ↓
[Consensus Voting]
  - 시간복잡도: 3/3 동의 → O(n²)
  - 최적화: 2/3 = 메모이제이션 / 1 = 알고리즘 자체 변경
       ↓
[Synthesis]
  핵심 동의: O(n²)
  Trade-off: 메모이제이션 (간단) vs DP 재구성 (성능 좋음)
```

### ✅ 결과 확인

```bash
git worktree list
# 일시적으로 만들어진 worktree 들 (council 끝나면 정리됨)
```

---

## 12. 변형 / 관련 프로젝트

| 프로젝트 | 용도 |
|---|---|
| [TechNickAI/openclaw-config](https://github.com/TechNickAI/openclaw-config) | Claude Code 용 OpenClaw 설정 — 영속 메모리 / 11 통합 스킬 / 4 자동 워크플로우 |
| [yoloshii/ClawMem](https://github.com/yoloshii/ClawMem) | Claude Code / Hermes / OpenClaw 공통 메모리 (Hooks + MCP + 하이브리드 RAG) |
| [ComposioHQ/secure-openclaw](https://github.com/ComposioHQ/secure-openclaw) | 500+ 앱 통합 + 보안 강화판 |
| [mergisi/awesome-openclaw-agents](https://github.com/mergisi/awesome-openclaw-agents) | 162 production-ready 에이전트 템플릿 |

ClawMem 셋업 (멀티 에이전트 메모리 공유):

```bash
npm install -g clawmem
clawmem init --share-with openclaw,hermes,claude-code
```

---

## 13. 트러블슈팅

### 데몬 안 도는 경우

```bash
openclaw doctor
# 또는
openclaw status
```

```bash
openclaw daemon stop
openclaw daemon start
```

### 채널 메시지 안 옴

```bash
openclaw doctor --channels
```

webhook URL / token / 권한 확인.

### 메모리 / 세션이 망가짐

```bash
ls ~/.openclaw/sessions/
# 세션 디렉토리 백업

# 데이터 리셋 (메모리 유지)
openclaw reset --keep-memory

# 완전 리셋 (위험)
openclaw reset --all
```

### npm install 권한 에러 (macOS)

```bash
# nvm 사용 권장
brew install nvm
nvm install 24
nvm use 24
npm install -g openclaw@latest
```

---

## 14. Hermes Agent 와의 비교

| 항목 | OpenClaw | [Hermes](./hermes-workshop.md) |
|---|---|---|
| 핵심 가치 | **셀프호스트 + 멀티채널 비서** | **자기 진화 학습 루프** |
| Live Canvas | ✅ 시각 워크스페이스 | ❌ |
| Wake word | ✅ macOS/iOS | ❌ (음성은 있음) |
| 자율 워크플로우 | ✅ 강력 (cron + 이벤트 트리거) | 일부 |
| 자기 스킬 진화 | ❌ (수동/플러그인) | ✅ 자동 |
| Claude Code 통합 | ✅ openclaw-claude-code 플러그인 (27 tools) | ✅ claude-code 스킬 |
| 영속 메모리 | ✅ | ✅ (더 능동적) |

둘 다 깔아 [ClawMem](https://github.com/yoloshii/ClawMem) 으로 메모리 공유하는 패턴도 있음.

---

## 15. 전체 흐름 요약

```
[설치]
  curl install.sh | bash
  openclaw onboard --install-daemon
   ↓
[모델 / 채널 셋업]
  텔레그램 / 디스코드 / 슬랙 등 추가
   ↓
[CLI 첫 대화]
  openclaw chat
   ↓
[스킬 추가]
  Gmail / GitHub / Calendar / Notion ...
   ↓
[자율 워크플로우 등록]
  - 일일 브리핑 (cron)
  - PR 알림 (webhook)
  - 1on1 의제 (weekly)
   ↓
[Live Canvas / 음성 / Wake word]
   ↓
(선택) [Sandbox 모드]
  그룹 채널은 docker 격리
   ↓
(선택) [Claude Code 플러그인]
  npm i -g @enderfga/openclaw-claude-code
  /ultraplan, /ultrareview, /council
   ↓
[데일리 사용]
  메신저 어디서든 같은 인격
```

수고하셨습니다 🦞

---

## 16. 다른 스택과 함께

| 조합 | 효과 |
|---|---|
| OpenClaw + [Hermes](./hermes-workshop.md) | 둘 다 메신저 비서, ClawMem 으로 메모리 공유 |
| OpenClaw + [Claude Code](../agents/claude-code-workshop.md) | OpenClaw가 비서, 코딩은 Claude Code 위임 |
| OpenClaw + [Superpowers](./superpowers-workshop.md) | OpenClaw 가 위임한 Claude Code 가 디시플린 따라 동작 |

---

## 17. 더 보기

- 레퍼런스: [`docs/extensions/openclaw.md`](../../docs/extensions/openclaw.md)
- 메인: https://github.com/openclaw/openclaw
- 공식 사이트: https://openclaw.ai/
- Claude Code 플러그인: https://github.com/Enderfga/openclaw-claude-code
- 호스팅: https://www.oneclaw.net/
- OpenClaw vs Hermes 비교: https://thenewstack.io/persistent-ai-agents-compared/
- ClawMem (메모리 공유): https://github.com/yoloshii/ClawMem
- awesome-openclaw-agents: https://github.com/mergisi/awesome-openclaw-agents
