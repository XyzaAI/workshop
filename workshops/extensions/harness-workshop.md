# Harness 핸즈온 워크샵

[Harness](../../docs/extensions/harness.md) (revfactory/harness, v1.2.0+) — Claude Code의 **메타 팩토리**. "이 도메인 작업을 하고 싶어" 라고 한 줄 던지면 그 도메인 맞춤 **에이전트 팀 + 스킬 묶음**을 자동으로 생성합니다.

이 워크샵에서는:
1. Agent Teams 활성화 + Harness 설치
2. **Pipeline** 패턴 — 데이터 수집 → 정제 → 리포트 자동 생성
3. **Fan-out/Fan-in** 패턴 — 여러 LLM 의견 병렬 수집 후 합성
4. **Producer-Reviewer** 패턴 — 코드 생성 + 보안 리뷰
5. **Expert Pool** 패턴 — 도메인 라우팅 (FE/BE/DB)
6. **Supervisor** / **Hierarchical Delegation** 차이 보기
7. dry-run 검증, 비교 분석
8. 생성된 결과물을 일반 Claude Code 사이클에서 사용

매 단계 ✅ 결과 확인 (생성된 `.claude/agents/`, `.claude/skills/` 파일들 직접 보기) 가 있습니다.

> 사전 학습: [Claude Code 워크샵](../agents/claude-code-workshop.md)
> 레퍼런스: [`docs/extensions/harness.md`](../../docs/extensions/harness.md)

---

## 0. 사전 준비

| 항목 | 확인 |
|---|---|
| Claude Code (최신) | `claude --version` |
| Anthropic 인증 | `/login` 상태 |
| Agent Teams 환경변수 | `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` (값 `1`) |

### Agent Teams 활성화

```bash
# 한 번만
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
source ~/.zshrc
```

또는 일회성:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

> ⚠️ 안 켜면 Harness 가 만든 팀이 **로딩 안 됨**.

---

## 1. 설치

Claude Code 안에서:

```
> /plugin marketplace add revfactory/harness
> /plugin install harness@harness
```

또는 직접 clone:

```bash
git clone https://github.com/revfactory/harness ~/.claude/skills/harness
```

Claude Code 재시작.

### ✅ 결과 확인

```
> /plugin
```

목록에 `harness v1.2.0+` 표시.

```bash
ls ~/.claude/plugins/harness/  # 또는 ~/.claude/skills/harness/
# 패턴 정의 / 템플릿 / SKILL.md 등
```

---

## 2. 빈 프로젝트 폴더

```bash
mkdir ~/Desktop/harness-demo
cd ~/Desktop/harness-demo
git init
echo "node_modules" > .gitignore
git add .gitignore
git commit -m "chore: init"
claude
```

---

## 3. 첫 Harness — Pipeline 패턴

가장 간단한 의존 순차 작업.

```
> build a harness for fetching daily Hacker News top 10, summarizing each story,
  and posting a digest to my email.
```

Harness 가 6 phase 자동 진행:

```
[Phase 1: Domain Analysis]
  - 작업: HN 크롤 → 요약 → 이메일
  - 의존: A → B → C 순차 (각 단계가 이전 결과에 의존)
  → Pipeline 패턴 선택

[Phase 2: Team Architecture]
  Pipeline:
    Stage 1: hn-fetcher
    Stage 2: story-summarizer
    Stage 3: digest-emailer

[Phase 3: Agent Definitions 생성 중...]
[Phase 4: Skills 생성 중...]
[Phase 5: 통합...]
[Phase 6: dry-run...]

✅ 완료. 다음 명령으로 사용 가능:
  > run hn-digest pipeline
```

### ✅ 결과 확인

```bash
ls .claude/agents/
# hn-fetcher.md
# story-summarizer.md
# digest-emailer.md

ls .claude/skills/
# fetch-hn-top/
# summarize-story/
# format-email-digest/
# send-email/

# 한 파일 열어보기
cat .claude/agents/hn-fetcher.md
```

내용 예:

```markdown
---
name: hn-fetcher
description: Use as the first stage of HN digest pipeline. Fetches top 10 stories from HN front page.
tools: Bash, Read
output_format: json
---

당신은 HN 데이터 수집 전문가입니다.

## 절차
1. https://news.ycombinator.com/news 에서 top 10 추출
2. 각 항목의 (id, title, url, score, comments_count) 를 JSON 으로 출력
3. 다음 stage (story-summarizer) 가 사용할 형식 정확히 준수

## 출력 형식
```json
{ "stories": [{ "id": 12345, "title": "...", ... }] }
```
```

### Pipeline 실행

```
> run hn-digest pipeline
```

3개 에이전트가 순차로 동작. 각 단계 결과를 다음으로 넘김.

```bash
git log --oneline | head
# feat(harness): hn-digest pipeline scaffold
# (사용자가 ship 하면) chore: digest of YYYY-MM-DD
```

---

## 4. Fan-out / Fan-in — 병렬 의견 수집

```
> build a harness that asks Claude, GPT, and Gemini the same coding question,
  collects their answers, and synthesizes a consensus answer.
```

Harness 분석:

```
[Phase 2: Team Architecture]
  Fan-out / Fan-in:
    fan-out: 3 워커 (claude-asker, gpt-asker, gemini-asker)
    fan-in: 1 합성기 (synthesizer)
```

### ✅ 결과 확인

```bash
ls .claude/agents/
# router.md
# claude-asker.md
# gpt-asker.md
# gemini-asker.md
# synthesizer.md

ls .claude/skills/
# call-claude/
# call-gpt-via-codex/
# call-gemini/
# synthesis-3-way/
```

### 실행

```
> run cross-llm-question "Why does this Python code have a deadlock?
  <code snippet>"
```

3개 에이전트가 **병렬 실행** → synthesizer 가 답변 비교 → 합의안 또는 차이 분석 출력.

```
[Result]
3개 모델 모두 동의: GIL + threading.Lock 순서가 데드락 원인
- Claude: 추가로 asyncio 사용 권장
- GPT: 추가로 timeout 옵션 제안
- Gemini: 동의
[Synthesis] 핵심 이슈: A. 부가: asyncio 전환 또는 timeout.
```

---

## 5. Producer-Reviewer — 생성 + 품질 게이트

```
> build a harness for writing API endpoints and automatically getting a security review.
```

```
[Phase 2]
  Producer-Reviewer:
    Producer: api-builder       (코드 생성)
    Reviewer: security-reviewer (OWASP/STRIDE 검토)
    Gate: pass/fail → 통과해야 다음으로
```

### ✅ 결과 확인

```bash
cat .claude/agents/security-reviewer.md
```

이 에이전트는 OWASP Top 10 + STRIDE 위협 모델 + 검증된 발견만 보고하도록 정의되어 있어야 함.

### 실행

```
> /api-builder /todos POST 엔드포인트 추가. body: {title, due_date}.
```

흐름:
```
[api-builder] 코드 생성
       ↓
[security-reviewer] 분석:
  - SQL Injection 가능성 (✗ 발견 → 차단)
  - XSS 위험 (✓ 안전)
  - 인증 누락 (✗ 발견 → 차단)
       ↓
[Gate FAIL]
api-builder 에 피드백 → 재시도
       ↓
[Gate PASS] → 코드 commit
```

### ✅ 결과 확인

```bash
git log --oneline | head
# feat(api): POST /todos with auth + parametrized query
# (review-driven 수정이 같은 commit 또는 별도)
```

---

## 6. Expert Pool — 도메인 라우팅

```
> build a harness for full-stack code questions where queries get routed to
  frontend, backend, database, or devops experts based on context.
```

```
[Phase 2]
  Expert Pool:
    Router: domain-classifier
    Experts:
      - frontend-expert  (React/Vue/CSS)
      - backend-expert   (Node/Python/Go)
      - database-expert  (SQL/NoSQL/migrations)
      - devops-expert    (k8s/ci/infra)
```

### ✅ 결과 확인

```bash
ls .claude/agents/
# domain-classifier.md
# frontend-expert.md
# backend-expert.md
# database-expert.md
# devops-expert.md
```

### 실행

```
> /ask 이 React useEffect 무한 루프 문제 어떻게 풀어?
```

흐름:
```
[domain-classifier] context 분석 → frontend
[frontend-expert] React 전문 답변
```

다른 질문:

```
> /ask Postgres에서 partial index 와 expression index 차이?
```

```
[domain-classifier] → database-expert
```

라우팅 정확도가 핵심 — 잘못 라우팅되면 다시 학습.

---

## 7. Supervisor — 중앙 디스패치

비슷해 보이지만 차이가 있는 패턴.

| Expert Pool | Supervisor |
|---|---|
| 라우터가 한 명에게 위임 | 디스패처가 **여러 명에게 동시** 분배 |
| 한 작업 = 한 전문가 | 한 큰 작업 = 여러 워커 병렬 |
| context-dependent | task-decomposition |

```
> build a harness for migrating a monolith to microservices.
  multiple services need to be extracted in parallel.
```

```
[Phase 2]
  Supervisor:
    supervisor: migration-coordinator
    workers (병렬):
      - auth-extractor
      - billing-extractor
      - notification-extractor
      - shared-types-extractor
```

### ✅ 결과 확인

```bash
cat .claude/agents/migration-coordinator.md
```

이 supervisor 가 작업을 분해 → workers 병렬 실행 → 결과 통합 → 검증.

---

## 8. Hierarchical Delegation — 재귀 분해

가장 무거운 패턴. **boss → manager → workers** 의 트리 구조.

```
> build a harness for writing a technical book on distributed systems.
  i need outline → chapters → sections → paragraphs.
```

```
[Phase 2]
  Hierarchical Delegation:
    boss: book-architect
      └─ manager: chapter-planner
            └─ worker: section-writer
                  └─ worker: paragraph-writer + citation-fetcher
```

### ✅ 결과 확인

```bash
ls .claude/agents/
# book-architect.md
# chapter-planner.md
# section-writer.md
# paragraph-writer.md
# citation-fetcher.md
```

### 실행 (예제는 매우 무거움 — 실제론 작은 도메인 추천)

```
> /book-architect "Distributed Systems: Theory + Practice"
```

시간 / 비용이 많이 들 수 있으니 **plan 만 만들고 멈춤** 모드도 있는지 확인.

---

## 9. dry-run / 검증

생성된 harness 가 실제로 도는지 확인:

```
> /harness-dry-run hn-digest
```

또는 모든 harness:

```
> /harness-validate
```

### ✅ 결과 확인

출력 예:

```
[hn-digest pipeline]
  ✅ Stage 1: hn-fetcher — schema valid
  ✅ Stage 2: story-summarizer — input/output match
  ✅ Stage 3: digest-emailer — env DEPS available
  ✅ End-to-end with synthetic input: completed in 8.3s

[cross-llm-question]
  ✅ Fan-out 3-way: tested
  ⚠️ synthesizer prompt was ambiguous on tie-breaking → review .claude/agents/synthesizer.md
```

경고는 무시 말고 직접 가서 다듬기.

---

## 10. 비교 분석 — harness 효과 측정

같은 작업을 **harness 적용 전 / 후** 로 돌려 차이 보기:

```
> /harness-compare hn-digest

[Without harness]
  prompt: "fetch HN top 10 and summarize..."
  - latency: 38s
  - quality (LLM judge): 6.4/10
  - variance (3 runs stdev): 1.8

[With hn-digest harness]
  - latency: 22s
  - quality: 8.7/10
  - variance: 0.4
```

저자 측정 (n=15) 으로는 **품질 +60%, 분산 -32%** 보고됨. 자기 도메인에서 검증 권장.

---

## 11. harness 나열 / 제거

### ✅ 등록된 harness 보기

```
> /harness-list

1. hn-digest         (Pipeline)
2. cross-llm-question (Fan-out/Fan-in)
3. api-builder       (Producer-Reviewer)
4. fullstack-expert  (Expert Pool)
5. monolith-migration (Supervisor)
6. book-author       (Hierarchical)
```

### 제거

```
> /harness-remove book-author
```

→ `.claude/agents/book-architect.md` 등 관련 파일 모두 삭제 (또는 archive 폴더로 이동).

---

## 12. 다른 워크플로우와 결합

생성된 harness 자산은 **표준 Claude Code 형식**이므로 다른 스킬팩과 자유롭게 결합.

### Superpowers + Harness

```
[Harness] 도메인 맞춤 팀 1회 생성
   ↓
[Superpowers] 그 위에 brainstorming → plan → execute 디시플린 강제
```

예: harness 가 만든 `api-builder` 가 발동하기 전에 superpowers 의 `brainstorming` 자동 발동 → 요구사항 합의 → harness 의 producer-reviewer 가 실행.

### gstack + Harness

```
[gstack] 풀 사이클 (CEO → Eng → QA → Ship)
   ↓
[Harness] gstack 의 일반 스킬을 우리 도메인 맞춤으로 대체
```

예: gstack의 `/qa` 는 일반 브라우저 테스트지만, harness 가 만든 `mobile-qa` 가 모바일 전용 흐름을 자동.

### GSD + Harness

```
[GSD] 큰 프로젝트의 phase 관리
   ↓
[Harness] 각 phase 의 execute 를 도메인 맞춤 팀으로
```

---

## 13. 트러블슈팅

### Agent Teams 비활성

```
Error: Agent Teams not enabled. Set CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

```bash
echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS
# 1 이어야 함
```

`~/.zshrc` / `~/.bashrc` 에 영속화 후 셸 재시작.

### 패턴 선택이 이상

Harness 가 잘못된 패턴을 골랐다면:

```
> 위 harness 다시 만들어. Pipeline 말고 Fan-out 으로.
```

도메인 설명을 좀 더 구체적으로 → 보통 한 번에 정확히 잡힘.

### dry-run 실패

```bash
cat .claude/agents/<failing-agent>.md
```

직접 열어 description / tools / output_format 확인. 모호한 description 이 가장 흔한 원인.

### 너무 많은 에이전트 → 관리 어려움

```
> /harness-list
> /harness-remove <오래된거>
```

또는 간단한 경우엔 harness 안 쓰고 그냥 슬래시 커맨드 1개로:

```bash
echo "..." > .claude/commands/my-command.md
```

---

## 14. 전체 흐름 요약

```
[설치]
  CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
  /plugin install harness@harness
   ↓
[도메인 설명]
  build a harness for <도메인>
   ↓
[6 phase 자동 진행]
  Domain Analysis → Architecture Design → Agent/Skill Generation
  → Integration → Validation
   ↓
[패턴 선택]
  Pipeline / Fan-out / Expert Pool / Producer-Reviewer
  / Supervisor / Hierarchical Delegation
   ↓
[생성물]
  .claude/agents/*.md
  .claude/skills/<name>/SKILL.md
   ↓
[검증]
  /harness-dry-run
  /harness-validate
  /harness-compare
   ↓
[일상 사용]
  생성된 슬래시 커맨드 / 자연어로 호출
   ↓
(필요 시) Superpowers / gstack / GSD 와 조합
```

수고하셨습니다 🎉

---

## 15. 더 보기

- 레퍼런스: [`docs/extensions/harness.md`](../../docs/extensions/harness.md)
- 메인: https://github.com/revfactory/harness
- 비교 도구 (런타임 결정성): Archon
- 함께 쓸 만한 워크샵:
  - [Superpowers](./superpowers-workshop.md)
  - [gstack](./gstack-workshop.md)
  - [GSD](./gsd-workshop.md)
