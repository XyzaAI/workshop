# oh-my-opencode

[opensoft/oh-my-opencode](https://github.com/opensoft/oh-my-opencode) — **#1 OpenCode 플러그인**. "Battery included" 형태로 OpenCode를 처음부터 풀장착시켜주는 메가팩.

3.0 stable. 1,400+ 커밋의 누적 다듬음. **"오픈코드의 oh-my-zsh"** 정도로 보면 됨.

> 주의: 프로젝트가 [`code-yeongyu/oh-my-openagent`](https://github.com/code-yeongyu/oh-my-openagent) (omo) 로 리브랜딩 중. plugin 설정 키는 점점 `oh-my-openagent` 가 권장되며, 기존 `oh-my-opencode` 키는 호환성으로 로딩됨 (warning 출력).

---

## 무엇을 해주나

기본 OpenCode는 미니멀해서 실전 쓰기엔 직접 셋업이 많음. oh-my-opencode는 다음을 한 번에:

### 1. 멀티 에이전트 오케스트레이션

- **Sisyphus** (메인 에이전트, Claude Opus 4.5 High 베이스)
- **Oracle** — 아키텍처 / 디버깅 전담
- **Frontend Engineer** — UI 작업
- **Librarian** — 문서 리서치

비동기 서브에이전트 (Claude Code의 Subagent와 유사) 가 작동.

### 2. 강력한 툴

- **LSP / AST-Grep** 통합 — 정밀 리팩터링
- **Todo Continuation Enforcer** — 시작한 작업 끝까지 가게
- **Comment Checker** — 과도한 주석 차단
- **Claude Code Compatibility Layer** — Claude Code 워크플로우/스킬 그대로

### 3. 큐레이트된 MCP 3종 기본 탑재

| MCP | 역할 |
|---|---|
| **Exa** | 웹 검색 |
| **Context7** | 공식 문서 액세스 |
| **Grep.app** | GitHub 코드 검색 |

### 4. 병렬 백그라운드 에이전트

여러 에이전트 동시 실행, 모델/프로바이더 분산.

---

## 설치

공식 권고: **"사람이 손으로 깔지 말고 LLM 에이전트한테 시켜라"**. 사람은 실수하니까.

OpenCode 또는 다른 에이전트를 띄운 뒤:

```
> https://github.com/opensoft/oh-my-opencode 의 README 보고 이 머신에 설치해줘
```

수동 설치를 굳이 하려면 npm:

```bash
npm install -g oh-my-opencode@latest
# 또는 (리브랜딩 후)
npm install -g oh-my-openagent@latest
```

---

## 변형 / 포크

- **[oh-my-opencode-slim](https://github.com/alvinunreal/oh-my-opencode-slim)** — 토큰 소비 큰 게 부담스러울 때. Slim/clean/fine-tuned fork.
- **[sodam-ai/oh-my-opencode](https://github.com/sodam-ai/oh-my-opencode)** — 또 다른 미러/포크.
- **[oh-my-opencode-dashboard](https://github.com/WilliamJudge94/oh-my-opencode-dashboard)** — 에이전트 추적용 대시보드.
- **awesome 모음**: [awesome-opencode/awesome-opencode](https://github.com/awesome-opencode/awesome-opencode).

---

## 언제 쓰면 좋나

**잘 맞음**:
- OpenCode를 메인으로 쓰는데 "Claude Code급 풍부함"을 원할 때
- MCP를 직접 셋업하기 귀찮을 때
- 멀티 에이전트 / 백그라운드 작업을 자주 굴릴 때

**덜 맞음**:
- 토큰가 민감 — 기본팩이 무거움 (slim fork 고려)
- 미니멀리즘 선호 — 너무 많은 게 깔림
- Claude Code를 메인으로 쓰는 경우 — 굳이 OpenCode를 추가로 깔 필요는 없음

---

## 참고

- 메인: https://github.com/opensoft/oh-my-opencode
- 리브랜딩(omo): https://github.com/code-yeongyu/oh-my-openagent
- Slim 포크: https://github.com/alvinunreal/oh-my-opencode-slim
- 본체 OpenCode: [`../agents/opencode.md`](../agents/opencode.md)
