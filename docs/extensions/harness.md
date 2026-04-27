# Harness

[revfactory/harness](https://github.com/revfactory/harness) (v1.2.0+) — Claude Code의 **"Team-Architecture Factory"** 플러그인. 도메인 설명을 받아 그 도메인에 맞는 **에이전트 팀 + 스킬 묶음**을 자동 생성합니다.

> 핸즈온 가이드: [`../../workshops/extensions/harness-workshop.md`](../../workshops/extensions/harness-workshop.md). 이 문서는 **레퍼런스**.

---

## 한 줄 정리

다른 스킬팩이 **"이런 스킬을 가져다 써라"** 라고 한다면, Harness는 **"네 도메인에 맞는 스킬을 자동으로 만들어준다"**.

Layer로 보면:

| Layer | 예 |
|---|---|
| L0 | Claude Code 본체 |
| L1 | 도구 (MCP, Bash, Edit, ...) |
| L2 | 스킬 / 서브에이전트 (Superpowers, gstack 의 결과물) |
| **L3 (Meta)** | **Harness — L2를 만드는 것** |

따라서 Superpowers/gstack과 **경쟁이 아니라 보완**. 보통은 Harness 로 한 번 만들어두고 그 결과물을 일상에 사용.

---

## 6가지 아키텍처 패턴

도메인 설명을 받아 다음 6가지 중 **자동으로 적합한 패턴 선택** 후 코드 생성.

| 패턴 | 사용 상황 | 그림 |
|---|---|---|
| **Pipeline** | 의존 순차 작업 | A → B → C → D |
| **Fan-out / Fan-in** | 독립 병렬 작업 후 합치기 | A → {B,C,D} → E |
| **Expert Pool** | 컨텍스트 따라 전문 에이전트 선택 | 라우터 → 전문가₁/₂/₃/... |
| **Producer-Reviewer** | 생성 + 품질 게이트 | Producer → Reviewer → ✓/✗ |
| **Supervisor** | 중앙 코디네이터가 분배 | Supervisor → Worker₁/₂/₃ |
| **Hierarchical Delegation** | top-down 재귀 분해 | Boss → Manager → Workers |

각 패턴별 적합 예:

- **Pipeline**: 데이터 파이프라인 (수집 → 정제 → 분석 → 리포트)
- **Fan-out/Fan-in**: 멀티 모델 비교, 병렬 검색
- **Expert Pool**: 도메인 다중 (FE / BE / DB / DevOps 질문 라우팅)
- **Producer-Reviewer**: 코드 생성 + 보안 리뷰 / 디자인 + QA
- **Supervisor**: 큰 프로젝트 디스패치
- **Hierarchical Delegation**: 매우 큰 작업 (책 쓰기, 큰 리팩터)

---

## 설치

### 1. Agent Teams 활성화 (필수)

Claude Code 의 실험 기능이라 환경변수 필요:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
# ~/.zshrc 또는 ~/.bashrc 에 영속화
```

### 2. 마켓플레이스 설치

Claude Code 안에서:

```
> /plugin marketplace add revfactory/harness
> /plugin install harness@harness
```

### 3. 또는 직접 clone

```bash
git clone https://github.com/revfactory/harness ~/.claude/skills/harness
```

---

## 동작 방식 (6단계 phase)

도메인 설명을 받으면 다음 phase 를 자동 진행:

```
1. Domain Analysis           — 도메인 / 작업 분석
2. Team Architecture Design  — 6 패턴 중 선택
3. Agent Definition          — .claude/agents/<name>.md 생성
4. Skill Generation          — .claude/skills/<name>/SKILL.md 생성
5. Integration & Orchestration — 에이전트 간 데이터 / 에러 흐름
6. Validation & Testing      — dry-run + 비교 테스트
```

---

## 사용 패턴

```
> build a harness for <도메인 설명>
```

예시:

```
> build a harness for deep research on AI papers
```

Harness 가:
- "Expert Pool" 패턴 선택 (논문 종류별 전문가)
- 5~7개 에이전트 정의 생성 (.claude/agents/)
- 각 에이전트의 스킬 (.claude/skills/) 생성
- 라우터 / 통합 / 에러 핸들링 코드 추가
- dry-run 으로 검증

---

## 생성되는 파일 구조

```
<project>/
└── .claude/
    ├── agents/
    │   ├── researcher-router.md     # 입력 → 적절한 전문가에 위임
    │   ├── theory-expert.md
    │   ├── empirical-expert.md
    │   ├── application-expert.md
    │   └── synthesizer.md           # 결과 합치기
    └── skills/
        ├── arxiv-deep-search/
        │   └── SKILL.md
        ├── paper-summarize/
        │   └── SKILL.md
        ├── citation-analyze/
        │   └── SKILL.md
        └── synthesis-report/
            └── SKILL.md
```

각 파일은 Claude Code 의 표준 형식이라 **harness 없이도 동작** — 즉 lock-in 이 없음.

---

## Progressive Disclosure

생성되는 스킬은 **컨텍스트 효율을 위한 progressive disclosure** 적용:

```markdown
---
name: arxiv-deep-search
description: Use when researching academic papers in depth. Pulls full text, citations, related work.
---

## 1단계 (가벼움)
검색어 받아 arXiv API 로 메타데이터만 조회. ~2K 토큰.

## 2단계 (필요 시)
사용자가 "더 자세히" 요청 시:
- 전체 논문 PDF 가져오기
- 인용 그래프 분석
~30K 토큰. 이 단계 가기 전 사용자 확인 권장.

## 3단계 (드물게)
관련 연구 자동 추적 (인용된 논문의 인용된 논문). ~100K 토큰.
이 단계는 사용자가 명시적으로 요청한 경우만.
```

→ 가벼운 작업엔 가벼운 스킬, 깊은 작업엔 단계적 expand.

---

## 검증 / Dry-run

생성 완료 후 자동 검증:

```
[harness] Validation phase
✅ Agent definitions parse correctly
✅ All skill descriptions are unambiguous
✅ No circular dependencies between agents
✅ Dry-run with synthetic input succeeded
✅ Output variance test (3 runs): consistent
```

또는 **비교 분석** — harness 적용 전/후를 같은 작업으로 돌려 차이 측정.

저자 측정 (n=15 작업):
- 평균 품질 점수 49.5 → 79.3 (**+60%**)
- 출력 분산 -32% (일관성 ↑)

> 측정 방식 / 작업 종류는 저자 보고 기준이라 자기 도메인에서 직접 검증 권장.

---

## 자주 쓰는 슬래시 커맨드

| 명령 | 동작 |
|---|---|
| `build a harness for <domain>` | 도메인 받아 풀 사이클 (자연어) |
| `/harness-dry-run` | 현재 harness 의 dry-run 검증 |
| `/harness-validate` | 정합성 검사 |
| `/harness-list` | 등록된 harness 목록 |
| `/harness-remove <name>` | 제거 |

(정확한 슬래시 명은 v1.2.0 기준이며 버전 따라 변경 가능. `/help harness` 로 확인.)

---

## 언제 안 맞나

- **이미 잘 잡힌 워크플로우** — 1인 / 단일 도메인 / 잘 짜인 슬래시 커맨드 몇 개로 충분
- **빠른 일회성 작업** — harness 생성 자체에 시간/토큰 듦
- **Agent Teams 미활성** 환경 — env 토글 안 켜진 곳에선 동작 안 함

---

## 다른 도구와의 관계

| 도구 | 역할 | Harness 와의 차이 |
|---|---|---|
| [Superpowers](./superpowers.md) | 디시플린 (TDD/디버깅) | 미리 만들어진 스킬을 사용. Harness 는 그런 스킬을 만듦 |
| [gstack](./gstack.md) | 풀 사이클 역할 (CEO/QA) | 고정 23 역할. Harness 는 도메인 맞춤 |
| [GSD](./gsd.md) | 단계 컨텍스트 관리 | 명령 셋이 정해져 있음. Harness 는 새 명령 셋을 생성 |
| Archon | 런타임 결정성 | 런타임 vs 아키텍처 — Archon 으로 결정성 + Harness 로 팀 구성, 둘 다 가능 |

조합:

```
Harness (1회) → 도메인 맞춤 팀/스킬 자동 생성
       ↓
Superpowers (상시) → 그 위에 디시플린 강제
       ↓
GSD (장기 작업) → 단계별 컨텍스트 관리
```

---

## 라이선스

Apache 2.0. README 한국어 / 일본어 / 영어 지원.

---

## 참고

- 메인: https://github.com/revfactory/harness
- 핸즈온 워크샵: [`../../workshops/extensions/harness-workshop.md`](../../workshops/extensions/harness-workshop.md)
- 비교 대상 (런타임 결정성): Archon
