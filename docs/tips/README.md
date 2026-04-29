# 활용 팁

도구 단위가 아니라 "이렇게 쓰면 좋다"는 노하우를 모은 폴더입니다.

대부분 Claude Code 기준으로 적었지만, 다른 에이전트에도 응용 가능한 패턴들입니다.

---

## 문서 목록

- [알림 / Notifications](./notifications.md) — 작업 끝났을 때 효율적으로 알림 받기
- [Hooks](./hooks.md) — 특정 이벤트에 자동으로 명령 실행
- [MCP Setup](./mcp-setup.md) — Model Context Protocol 서버 연결
- [생산성 팁](./productivity.md) — 단축키 / 세션 / 컨텍스트 관리
- [Remote 작업 환경](./remote.md) — 서버 + SSH + 모바일로 어디서든 에이전트 접속

---

## 공통 원칙

### 1. **에이전트가 직접 실행하게 둬라**

처음에는 모든 명령을 직접 치고 싶지만, 권한 모드 / `permissions.allow` 를 잘 잡으면 에이전트가 lint/test/git 같은 반복 작업을 알아서 처리합니다. 이게 생산성 차이의 90%.

### 2. **세션은 짧고 명확하게**

긴 대화는 토큰을 낭비하고 모델 혼동도 늘어남. 작업 단위로 끊고 `/clear` 또는 새 세션.

### 3. **메모리(CLAUDE.md/GEMINI.md) 적극 활용**

같은 설명을 매번 하는 게 보이면 메모리에 박을 신호.

### 4. **자동화는 점진적으로**

처음부터 hooks 잔뜩 박으면 디버깅 지옥. 같은 작업을 3번 이상 반복할 때 자동화 검토.

### 5. **모델은 작업 성격으로 선택**

- 추론 무거운 / 큰 변경 → Opus, o3, Gemini Pro
- 단순 반복 / TDD 루프 → Sonnet, Haiku, grok-code-fast
- 토큰가 무서울 때 → 로컬 (Ollama + Hermes/Llama)
