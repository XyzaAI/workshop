# Hermes Agent

[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) — Nous Research가 만든 **자기 개선 AI 에이전트**. "The agent that grows with you"가 슬로건.

여타 에이전트와의 핵심 차이: **빌트인 학습 루프** — 작업하면서 스킬을 만들고, 사용 중에 그 스킬을 개선하고, 과거 대화를 검색해 자신을 점점 더 사용자에 맞춰갑니다.

---

## 핵심 컨셉

```
경험 → 스킬 자동 생성 → 사용 중 개선 → 영속 메모리 → 사용자 모델 갱신
   ↑                                                     ↓
   ←─────────────────── 다음 세션에 이어짐 ──────────────────
```

다른 에이전트들은 "세션 = 휘발성"인 반면, Hermes는 **세션 간 지식이 누적**됩니다. ChatGPT의 "메모리"보다 훨씬 적극적인 자체 진화 모델.

---

## 설치

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

지원 OS: Linux, macOS, WSL2, **Android (Termux 포함)**.

설치 후:
```bash
hermes
```

---

## 주요 기능

### 1. 자율 스킬 생성 / 진화

복잡한 작업을 처리하면 그 절차를 **스킬로 박제**해두고, 다음에 비슷한 일이 오면 그 스킬을 호출 — 사용하면서 더 다듬어짐.

### 2. 영속 메모리 + 자기 nudge

특정 시점에 "이건 기억해둘 가치 있어" 라고 스스로 판단해서 영속화. 단순 RAG가 아니라 **에이전트가 능동적**.

### 3. 크로스 세션 검색

과거 대화 풀텍스트 인덱싱 + LLM 요약. "지난 달에 그 라이브러리 추천했었지?" 같은 질문에 직접 답.

### 4. "어디에서나" 접근

CLI는 시작점일 뿐. **Telegram / Discord / Slack / WhatsApp / Signal** 등에서 동일 인격이 응답.

### 5. 모델 무관

Nous Portal, OpenRouter, NVIDIA NIM, OpenAI, 자체 엔드포인트 등 — 코드 변경 없이 모델 스위칭.

### 6. 배포 유연성

$5 VPS / GPU 클러스터 / 서버리스 다 됨. 백엔드는 Docker / SSH / Daytona / Modal 등 6종 지원.

---

## Claude Code와의 관계

Hermes 자체는 **독립된 에이전트 런타임**이지만, Claude Code를 **도구로 호출**할 수 있습니다.

```
사용자 → Hermes (스킬, 메모리, 메신저)
                 ↓ (코딩 작업 위임)
              Claude Code (실제 파일 수정 / 빌드 / 테스트)
```

Hermes는 "장기 비서", Claude Code는 "즉시 실행 코더" 같은 역할 분담이 자연스럽습니다.

### Claude Code에 더 깊이 통합된 변형

- [`AlexAI-MCP/hermes-CCC`](https://github.com/AlexAI-MCP/hermes-CCC) — Hermes를 **Claude Code 안으로** 옮긴 포팅. 46개 네이티브 스킬, OAuth 불필요, 외부 프로세스 없음.
- [`sypsyp97/claude-hermes`](https://github.com/sypsyp97/claude-hermes) — 가벼운 OSS, Claude Code 안에서 동작.
- [`yoloshii/ClawMem`](https://github.com/yoloshii/ClawMem) — Claude Code / Hermes / OpenClaw용 온디바이스 메모리 레이어.

---

## 어떻게 쓰면 좋나

**잘 맞는 시나리오**:
- 메신저로 시키는 영구 비서 (출근 중 Telegram, 집에서는 CLI)
- 같은 도메인 작업을 반복 — 스킬이 점점 똑똑해짐
- 자주 까먹는 컨텍스트를 자동 보존 (예: "내 동료 X는 Y 좋아함")

**덜 맞는 시나리오**:
- 1회성 작은 작업 (오버스펙)
- 강한 격리가 필요한 보안/규제 환경 (영속 메모리는 위협 모델 추가)

---

## awesome 모음

[`0xNyk/awesome-hermes-agent`](https://github.com/0xNyk/awesome-hermes-agent) — 스킬 / 도구 / 통합 / 리소스 큐레이트.

---

## 라이선스

MIT.

---

## 참고

- 메인: https://github.com/NousResearch/hermes-agent
- 공식 사이트: https://hermes-agent.nousresearch.com
- 스킬 허브: https://hermes-agent.nousresearch.com/docs/skills/
- OpenClaw vs Hermes 비교: https://thenewstack.io/persistent-ai-agents-compared/
