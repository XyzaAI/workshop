# XYZA.ai 확장 도구 비교 로그

> GSD, gstack, Superpowers 세 가지 Claude Code 확장 도구로 각각 다른 제품을 구현하며 관찰한 워크플로우 비교 기록.

---

## 1. XYZA.ai 프로젝트 개요

### 회사 소개

XYZA.ai는 AI 개발자 생산성 도구, Agent 운영 플랫폼, 생태계 마켓플레이스, 도입 컨설팅/교육을 한 흐름으로 묶어 제공하는 AI 전문 회사다. 핵심 철학은 "Don't panic" -- 복잡성과 불확실성 속에서 감정이 아니라 루프(질문 -> 실험 -> 검증 -> 확장)로 답을 찾는다.

기존 AI 도입이 "도구 붙이기"에 머무르는 반면, XYZA는 **구조(조직 + 데이터 설계)** 중심으로 접근하며, 전략 문서만 전달하는 것이 아니라 **같이 만들고 운영까지** 함께한다.

### 4대 제품

| 제품 | 설명 | 도메인 |
|------|------|--------|
| **ATM (Automata)** | AI 설정/워크플로우 관리 CLI. skills/agents/commands를 선언적으로 관리 (`init` -> `migrate` -> `apply`) | automata.xyza.ai |
| **Golem Company** | AI Agent 고용/관리 플랫폼. 이름, 근무시간, 비용, 이슈 추적, 옵저빌리티를 HR 메타포로 제공 | golem.xyza.ai |
| **MCP/Plugin Market** | MCP 서버/플러그인 공유/배포/설치 마켓플레이스 | market.xyza.ai |
| **AX 컨설팅** | AI Experience 컨설팅. 기업 AI 도입 전략, 워크플로우 설계 (진단 -> 설계 -> 구현 -> 교육) | - |

### 기술 스택

| 영역 | 기술 |
|------|------|
| 프론트엔드 | React 19 |
| 백엔드 API | NestJS |
| 모노레포 | Turbo + Yarn 4.6.0 |
| 인프라 | Kubernetes |
| 앱 구조 | `apps/xyza-front`, `apps/core-api`, `apps/xyza-console-front` 등 6개 앱 |
| 패키지 관리 | `packages/` 공유 패키지 |

---

## 2. 확장 도구별 구현 매핑

| 도구 | 구현 대상 | 구현 범위 | 선정 이유 |
|------|----------|----------|----------|
| **GSD** | ATM CLI (`init` 명령) | TypeScript CLI + Commander.js + JSON 저장 | GSD는 `.planning/` 산출물 기반 스펙 드리븐 개발 시스템. CLI 도구는 명확한 Phase 분리(데이터 레이어 -> CLI 인터페이스 -> 마무리)가 자연스럽고, discuss -> plan -> execute 파이프라인이 커맨드 단위 구현에 잘 맞는다. |
| **gstack** | XYZA.ai 랜딩 페이지 | 정적 HTML/CSS, 반응형, 이메일 대기자 폼 | gstack은 CEO/Designer/Eng/QA 역할 기반 사이클. 랜딩 페이지는 비즈니스(CEO 리뷰), 디자인(시안 비교), 엔지니어링(성능 예산), QA(브라우저 테스트)를 모두 거쳐야 해서, 역할별 리뷰 구조가 가장 효과적이다. |
| **Superpowers** | Golem Company 코어 | Markdown -> HTML 정적 사이트 빌더 CLI (데이터 모델 + 빌드 파이프라인) | Superpowers는 검증 강제 디시플린 프레임워크. 데이터 모델과 빌드 파이프라인은 "동작하는지" 검증이 핵심이고, systematic-debugging + verification-before-completion이 파이프라인 신뢰성을 보장한다. |

### 왜 이 조합인가?

- **CLI 도구(ATM) + GSD**: CLI는 "입력 -> 처리 -> 출력"이 명확하다. GSD의 discuss-phase가 기술 선택(tsx vs ts-node, 저장 경로 등)을 미리 확정하고, execute-phase가 phase 단위로 atomic commit을 만든다.
- **웹 UI(랜딩) + gstack**: 웹 페이지는 이해관계자가 많다. gstack의 `/office-hours`가 비즈니스 질문을 강제하고, `/design-shotgun`이 시안 비교를, `/qa`가 실제 브라우저 테스트를 돌린다.
- **데이터 모델(Golem 코어) + Superpowers**: 데이터 파이프라인은 중간 단계마다 "이 출력이 정확한가?"를 검증해야 한다. Superpowers의 task별 검증 기준 + Phase별 종료 기준이 파이프라인 정확성을 보장한다.

---

## 3. 워크플로우 비교 (핵심)

### 3.1 계획 방식

| 축 | GSD | gstack | Superpowers |
|----|-----|--------|-------------|
| **계획 산출물** | `.planning/PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `N-CONTEXT.md`, `N-RESEARCH.md` | `DESIGN.md` (문제 정의 + CEO/Eng/Design 리뷰 통합) | `.plan.md` (task 단위 체크리스트) |
| **계획 트리거** | `/gsd-new-project` -> `/gsd-discuss-phase` -> `/gsd-plan-phase` | `/office-hours` -> `/autoplan` | `/brainstorm` -> `/write-plan` |
| **결정 구조** | Phase별 독립 컨텍스트 문서. 모호한 영역을 discuss에서 질문으로 확정 | 강제 질문(forcing questions) + 역할별 리뷰(CEO/Eng/Design). 각 역할이 자기 영역만 평가 | 브레인스토밍 인터뷰. 작은 청크로 나눠 받고, 다음으로 넘어가기 전 사용자 승인 |
| **리서치** | 자동 병렬 리서치 에이전트 (도메인 조사, 라이브러리 비교) -> `.planning/research/` | Eng 리뷰가 기술 제약을 걸고, Design 리뷰가 구체 스펙 지정 | 브레인스토밍 중 숨은 요구사항 발굴 |
| **계획 검증** | plan-checker가 plan 품질 자동 검증, 통과 못하면 재작성 루프 | CEO가 스코프, Eng가 기술 타당성, Design이 비주얼 품질을 각각 검증 | plan 자체가 검증 기준을 포함. task마다 "동작 확인 방법" 명시 |

**핵심 차이**: GSD는 **문서 계층**(PROJECT -> REQUIREMENTS -> ROADMAP -> CONTEXT)으로 결정을 쌓고, gstack은 **역할 리뷰**(CEO/Eng/Design)로 다면 평가하고, Superpowers는 **대화형 발굴**(brainstorm)로 요구사항을 끌어낸다.

### 3.2 실행 방식

| 축 | GSD | gstack | Superpowers |
|----|-----|--------|-------------|
| **실행 단위** | Phase -> Plan -> Wave (병렬 가능) | 일반 Claude Code 흐름 (gstack 전용 빌드 커맨드 없음) | Phase -> Task (2~5분 단위, fresh 서브에이전트) |
| **파이프라인** | discuss -> plan -> execute (각 Phase마다 반복) | Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect (7단계 사이클) | Brainstorm -> Write-plan -> Execute-plan (3단계) |
| **병렬성** | Wave 기반 병렬 실행. 독립 plan은 동시에 fresh 200K 컨텍스트에서 | 없음 (Build는 일반 코딩) | `dispatching-parallel-agents`로 독립 task 병렬 dispatch |
| **컨텍스트 관리** | 매 실행이 fresh 컨텍스트. plan이 모든 정보를 담아야 함 | DESIGN.md가 단일 진실 소스(single source of truth) | 서브에이전트가 fresh 200K 컨텍스트. plan + spec이 컨텍스트 전달 수단 |
| **커밋 전략** | task마다 atomic commit (`feat(phase-1.1): ...`) | Build 단계에서 기능 단위, Review에서 fix 단위 | Phase 경계에서 의미 있는 커밋 |
| **자동화 수준** | `/gsd-autonomous`로 전체 phase 자동 진행 가능 | 각 단계를 명시적으로 호출 | `/execute-plan`이 plan 기반 자동 진행 |

**핵심 차이**: GSD는 **plan 문서가 실행을 드라이브**하고, gstack은 **역할이 사이클을 드라이브**하고, Superpowers는 **서브에이전트가 task를 드라이브**한다.

### 3.3 검증 방식

| 축 | GSD | gstack | Superpowers |
|----|-----|--------|-------------|
| **주요 검증 수단** | `/gsd-verify-work` (대화형 UAT) | `/qa` (실제 Chromium 브라우저 테스트) | `verification-before-completion` (자동 발동) |
| **검증 시점** | Phase 완료 후 사용자에게 y/n 확인 | Build 후 별도 QA 단계 | 모든 task 완료 시, Phase 경계, 최종 전달 전 |
| **검증 범위** | REQUIREMENTS.md의 UAT 체크리스트 기준 | 스크린샷 캡처, 버그 리포트, 자동 regression test 생성 | 빌드 실행, 테스트 실행, 엣지 케이스, dead code 검출 |
| **실패 처리** | 실패 시 debug 에이전트 dispatch -> fix plan 생성 -> 다시 execute | 버그 fix -> atomic commit -> 재검증 | `systematic-debugging` 자동 발동 (가설 3개+ -> 검증 -> 근본 원인) |
| **보안 검증** | `/gsd-secure-phase` | `/cso` (OWASP Top 10 + STRIDE 위협 모델) | 없음 (별도 도구 필요) |
| **성능 검증** | 없음 (별도) | `/benchmark` (Core Web Vitals, Lighthouse) | 없음 (별도) |

**핵심 차이**: GSD는 **사용자 수락 테스트(UAT)**로 "약속한 것을 만들었는가"를 확인하고, gstack은 **실제 브라우저**로 "사용자에게 보이는 대로 동작하는가"를 확인하고, Superpowers는 **코드 레벨 검증**으로 "빌드되고, 테스트 통과하고, dead code 없는가"를 확인한다.

### 3.4 산출물 비교

| 산출물 유형 | GSD | gstack | Superpowers |
|-----------|-----|--------|-------------|
| **프로젝트 정의** | `PROJECT.md` | `DESIGN.md` (문제 정의 섹션) | 브레인스토밍 대화 내 합의 |
| **요구사항** | `REQUIREMENTS.md` (FR/NFR/UAT) | `DESIGN.md` (성공 기준 + 범위 밖) | `.plan.md` (검증 기준 포함) |
| **로드맵** | `ROADMAP.md` (Phase 목록) | 없음 (단일 사이클) | `.plan.md` (Phase + Task 목록) |
| **기술 결정** | `N-CONTEXT.md`, `STATE.md` decisions | `DESIGN.md` Eng 리뷰 섹션 | 없음 (대화 내 암묵적) |
| **리서치** | `N-RESEARCH.md`, `research/` | 없음 | 없음 |
| **실행 계획** | `N-N-PLAN.md` (XML 구조) | 없음 (DESIGN.md가 암묵적 계획) | `.plan.md` (체크리스트) |
| **실행 결과** | `N-N-SUMMARY.md`, `N-VERIFICATION.md` | 코드 파일 + git log | 커밋 + plan 체크 표시 |
| **검증 결과** | `N-UAT.md` | `.gstack/qa-runs/`, 스크린샷 | 화면 출력 (파일 미생성) |
| **상태 추적** | `STATE.md` (위치 + 결정 로그) | 없음 | `.plan.md` 체크 상태 |
| **회고** | 없음 (별도) | `/retro`, `/learn` -> `~/.gstack/learnings/` | 없음 |

**산출물 밀도**: GSD > gstack > Superpowers. GSD는 phase당 5~7개 마크다운 파일을 생성한다. gstack은 DESIGN.md 하나에 집중한다. Superpowers는 `.plan.md` 하나와 코드만 남긴다.

### 3.5 강점 영역

| 프로젝트 유형 | 가장 유리한 도구 | 이유 |
|-------------|---------------|------|
| **CLI 도구** | GSD | Phase 분리가 자연스러움 (데이터 레이어 -> 인터페이스 -> 마무리). discuss에서 기술 선택 확정. atomic commit으로 깔끔한 git history. |
| **웹 UI / 랜딩 페이지** | gstack | 역할별 리뷰(CEO/Eng/Design)가 다면 평가. `/design-shotgun`으로 시안 비교. `/qa`로 실제 브라우저 테스트. `/benchmark`로 성능 측정. |
| **데이터 모델 / 파이프라인** | Superpowers | task별 검증 기준이 파이프라인 정확성 보장. systematic-debugging이 중간 단계 실패를 근본 원인까지 추적. verification-before-completion이 "동작한다"의 증거를 강제. |
| **보안 민감 프로젝트** | gstack | `/cso`가 OWASP + STRIDE를 자동 감사. 17개 false positive 제외 룰로 신뢰도 높은 결과. |
| **대규모 다중 Phase** | GSD | 워크스트림/워크스페이스로 병렬 작업. 마일스톤 audit + gap 분석. session pause/resume으로 장기 프로젝트 관리. |
| **빠른 프로토타이핑** | Superpowers | 브레인스토밍 + 계획이 ~5분. 서브에이전트가 병렬 실행. 작은 프로젝트에서 오버헤드가 가장 적음. |

---

## 4. 조합 전략

### 4.1 GSD + gstack + Superpowers 시너지

세 도구는 각각 다른 축을 강제하기 때문에, 함께 쓰면 보완 관계가 형성된다.

| 도구 | 강제하는 축 | 다른 도구에 없는 것 |
|------|-----------|-------------------|
| **GSD** | 단계별 컨텍스트 (산출물) | `.planning/` 아티팩트 누적, 마일스톤 관리, session pause/resume |
| **gstack** | 역할별 리뷰 (이해관계자) | CEO/Eng/Design/QA/CSO 다면 평가, 실제 브라우저 테스트, 성능 벤치마크 |
| **Superpowers** | 코드 검증 디시플린 | 자동 발동 스킬 (brainstorming 강제, debugging 강제, completion 검증 강제) |

### 조합 시너지 공식

```
역할(gstack) + 디시플린(Superpowers) + 단계 컨텍스트(GSD)
```

- **gstack이 "누가 검토하는가"를 정의**: CEO가 스코프를, Eng가 기술을, Design이 비주얼을 검토
- **Superpowers가 "어떻게 검증하는가"를 강제**: brainstorming 건너뛰기 방지, completion 전 증거 필수
- **GSD가 "무엇을 남기는가"를 보장**: 모든 결정이 `.planning/`에 마크다운으로 기록되어 추적 가능

### 4.2 XYZA.ai 같은 다중 제품 프로젝트 배분 전략

| 제품/영역 | 추천 도구 | 이유 |
|----------|----------|------|
| **ATM CLI** (init/migrate/apply) | GSD 메인 | 명령별 Phase 분리, 기술 결정 discuss, atomic commit |
| **XYZA.ai 랜딩 페이지** | gstack 메인 | 이해관계자 리뷰, 시안 비교, 브라우저 QA |
| **Golem Company 코어** (Agent 모델/옵저빌리티) | Superpowers 메인 | 데이터 모델 검증, 파이프라인 정확성, 디버깅 디시플린 |
| **MCP Market** (검색/설치 UI) | gstack + Superpowers | 웹 UI(gstack QA) + API 정확성(Superpowers 검증) |
| **core-api** (NestJS 백엔드) | GSD + Superpowers | Phase별 엔드포인트 구현(GSD) + 통합 테스트 검증(Superpowers) |
| **Console / Back-office** | gstack | 내부 도구지만 다면 리뷰(Eng + Design)가 품질 보장 |

### 4.3 실무 조합 패턴

#### 패턴 A: 전체 사이클 조합

```
GSD /gsd-new-project       -> 프로젝트 구조 + 요구사항 + 로드맵
gstack /office-hours        -> 비즈니스 질문 보강 (DESIGN.md)
GSD /gsd-discuss-phase N    -> Phase별 기술 결정
Superpowers /brainstorm     -> 구현 전 요구사항 최종 검증
GSD /gsd-plan-phase N       -> 실행 계획 (PLAN.md)
Superpowers /execute-plan   -> 서브에이전트 실행 + 자동 검증
gstack /qa                  -> 브라우저 QA
gstack /cso                 -> 보안 감사
GSD /gsd-verify-work N      -> UAT
GSD /gsd-ship N             -> PR
```

#### 패턴 B: 역할 분담 (팀 규모별)

| 팀 규모 | 추천 |
|---------|------|
| 1인 | Superpowers 단독 (오버헤드 최소) |
| 2~3인 | gstack + Superpowers (역할 + 디시플린) |
| 4인+ | GSD + gstack + Superpowers (전체 스택) |

---

## 5. 관찰과 교훈

### 5.1 세 도구의 공통점: "코딩 전에 생각하기"를 강제

세 도구 모두 **"바로 코드로 점프하는 것"을 막는다**. 방법만 다르다.

| 도구 | "코딩 전 생각"을 어떻게 강제하는가 |
|------|-------------------------------|
| GSD | `/gsd-discuss-phase`에서 모호한 영역을 질문으로 확정한 뒤에만 `/gsd-plan-phase` 가능. plan 없이 execute 불가. |
| gstack | `/office-hours`에서 강제 질문 6개를 통과해야 DESIGN.md 생성. Plan 없이 Build로 넘어갈 수 없음. |
| Superpowers | 모호한 요청을 던지면 brainstorming 스킬이 자동 발동. 코드 작성 전 명확화 질문 시작. |

공통 효과:
- 브레인스토밍/디스커버리에 쓴 ~5분이 실행 중 백트래킹 30분을 절약
- "내가 뭘 원하는지"를 사용자 자신도 깨닫게 됨
- 결정의 이유가 기록되어 "왜 이렇게 했지?" 추적 가능

### 5.2 세 도구의 핵심 차이: 무엇을 강제하느냐

| 강제 대상 | GSD | gstack | Superpowers |
|----------|-----|--------|-------------|
| **산출물** | `.planning/`에 phase마다 5~7개 문서. 결정, 리서치, 계획, 검증이 모두 파일로 남음 | `DESIGN.md` 하나에 집중. 이해관계자 의견이 한 문서에 통합 | `.plan.md` 하나. 나머지는 코드와 커밋으로 |
| **역할** | 없음 (도구가 역할 구분 안 함) | CEO, Eng Manager, Designer, QA, CSO, SRE 등 역할이 각자 검토 | 없음 (스킬이 자동 발동) |
| **검증** | UAT 체크리스트 기반 사용자 수락 | 실제 브라우저 + Lighthouse + OWASP/STRIDE | 빌드/테스트/에지케이스/dead code를 코드 레벨에서 자동 검증 |

### 5.3 실무 팁

1. **작은 프로젝트에서 GSD는 과잉일 수 있다.** `.planning/` 파일이 실제 코드보다 많아지는 경우가 있다. 1~2 phase짜리 작업이면 `/gsd-fast`나 `/gsd-quick`을 쓰거나, Superpowers 단독으로 충분하다.

2. **gstack의 진짜 가치는 Build가 아니라 Think와 Test다.** Build 단계는 일반 Claude Code와 동일하지만, `/office-hours`의 강제 질문과 `/qa`의 브라우저 테스트가 차별점이다. 이 두 단계만 빌려 써도 효과가 크다.

3. **Superpowers의 자동 발동이 때로는 거추장스럽다.** 한 줄 fix에도 brainstorming이 강제되면 생산성이 떨어진다. CLAUDE.md에 "한 줄 변경/typo fix는 brainstorming 생략"을 명시하면 해결된다.

4. **세 도구를 동시에 쓸 때 충돌 관리가 필요하다.** GSD의 `.planning/`과 Superpowers의 `.plan.md`가 "계획의 진실 소스"를 두고 경쟁할 수 있다. 원칙을 정해야 한다: GSD가 프로젝트 레벨 계획을, Superpowers가 task 레벨 계획을 담당하는 식으로.

5. **"생각이 비싼 부분이어야지, 코딩이 비싼 부분이면 안 된다."** gstack 데모에서 관찰한 바와 같이, Think+Plan이 Build보다 오래 걸리는 것이 올바른 비율이다. 세 도구 모두 이 비율을 자연스럽게 만든다.

6. **검증 방식은 제품 유형에 맞춰야 한다.** CLI 도구는 UAT(GSD)가, 웹 UI는 브라우저 테스트(gstack)가, 데이터 파이프라인은 코드 레벨 검증(Superpowers)이 가장 효과적이다. 하나의 검증 방식을 모든 제품에 적용하면 빈틈이 생긴다.
