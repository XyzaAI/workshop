# GSD (Get Shit Done)

[gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) — TÂCHES가 만든 **메타 프롬프팅 / 컨텍스트 엔지니어링 / 스펙 드리븐 개발** 시스템. Claude Code 2.1.88+, OpenCode, Gemini CLI, Codex, Cursor 등 12개 런타임 지원.

50K+ stars. 핵심은 **컨텍스트 부패 방지** — 각 단계의 산출물을 마크다운으로 영속화해서 매 작업이 신선한 컨텍스트로 시작하게 함.

> 핸즈온 가이드는 [`../../workshops/extensions/gsd-workshop.md`](../../workshops/extensions/gsd-workshop.md) 참고. 이 문서는 **레퍼런스**.

---

## 핵심 사이클

```
new-project → discuss-phase → plan-phase → execute-phase → verify-work → ship
   ↓             ↓               ↓             ↓               ↓           ↓
PROJECT.md   1-CONTEXT.md   1-RESEARCH.md  1-N-SUMMARY.md   1-UAT.md     PR
REQUIREMENTS                1-N-PLAN.md    1-VERIFICATION   git tag
ROADMAP                                    + 자동 commit
```

각 phase의 산출물은 `.planning/` 에 누적. 다음 phase의 컨텍스트가 됨. 메인 컨텍스트는 **30~40% 사용률 유지**.

---

## 설치

```bash
# 가장 간단 (latest)
npx get-shit-done-cc@latest

# 토큰 절감 모드 (--minimal)
npx get-shit-done-cc@latest --minimal
```

글로벌 또는 프로젝트 단위 선택. 설치 후 Claude Code 재시작하면 슬래시 커맨드들이 인식됩니다.

---

## 명령어 전체 (70+개) — 카테고리별 상세

### 🟢 핵심 워크플로우

#### `/gsd-new-project [--auto]`
**언제**: 새 프로젝트 시작 또는 기존 코드베이스를 GSD로 재초기화.

**동작**:
1. 인터랙티브 인터뷰 (목표/제약/기술 선호/엣지 케이스)
2. 병렬 리서치 에이전트 dispatch (도메인 조사)
3. v1/v2 요구사항 + 스코프 추출
4. 페이즈 단위 로드맵 생성
5. 사용자 승인

**생성**: `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, `.planning/research/`

#### `/gsd-discuss-phase [N] [--auto] [--analyze] [--chain]`
**언제**: phase 계획 전, 디자인 선호도 캡처.

**동작**: phase의 모호한 영역(레이아웃, API 형식, 에러 처리 등)을 식별 → 타깃 질문 → 결정사항을 컨텍스트 문서로.
- `--analyze`: 트레이드오프 분석 추가
- `--chain`: plan + execute 까지 자동 진행

**생성**: `{N}-CONTEXT.md`

#### `/gsd-plan-phase [N] [--auto] [--reviews]`
**언제**: 실행 전 atomic task plan 생성.

**동작**:
1. CONTEXT.md 기반으로 구현 접근법 리서치
2. 2~3개 atomic plan을 XML 구조로 (각각 fresh context에서 실행 가능)
3. plan-checker 가 phase 목표 달성 검증
4. 통과까지 반복 루프
- `--reviews`: 코드베이스 리뷰 결과 먼저 로드

**생성**: `{N}-RESEARCH.md`, `{N}-1-PLAN.md`, `{N}-2-PLAN.md`, ...

#### `/gsd-execute-phase <N>`
**언제**: phase의 모든 plan을 병렬 wave로 실행.

**동작**:
1. plan을 의존 순서로 wave 그룹핑
2. 독립 plan은 병렬 (각 200K fresh context)
3. 의존 plan은 dependency 끝난 뒤 순차
4. 태스크당 atomic git commit
5. 모든 코드가 phase 약속과 일치하는지 검증
6. 완료 리포트

**예시**:
- Wave 1: User Model + Product Model (병렬)
- Wave 2: Orders API + Cart API (병렬, 모델 의존)
- Wave 3: Checkout UI (두 API 의존)

**생성**: `{N}-{T}-SUMMARY.md`, `{N}-VERIFICATION.md` + git commits

#### `/gsd-verify-work [N]`
**언제**: phase 산출물의 manual UAT.

**동작**: testable deliverable 추출 → "X 가능한가요? Yes/no" 질문 → 실패 시 debug 에이전트가 근본 원인 진단 + fix plan 생성. 통과 시 ship 준비 완료 마크.

**생성**: `{N}-UAT.md`, fix plan (있다면)

#### `/gsd-ship [N] [--draft]`
**언제**: 검증 끝난 phase의 PR 생성.

**동작**: PR body 자동 생성, 변경사항/테스트 노트 포함. `.planning/` 제외 필터 적용.
- `--draft`: ready-for-review 대신 draft PR

**생성**: 원격 PR

#### `/gsd-next`
**언제**: 다음 단계 뭔지 모를 때.

**동작**: 현재 상태 자동 감지 → 적절한 다음 명령 자동 실행 (discuss → plan → execute → verify → ship).

#### `/gsd-fast <text>`
**언제**: 사소한 ad-hoc 작업, 풀 플래닝 불필요.

**동작**: 플래닝 건너뛰고 바로 실행. atomic commit 보장.

#### `/gsd-quick [--full] [--validate] [--discuss] [--research]`
**언제**: `/gsd-fast`보단 크고 phase 만들 정도는 아닌 작업.

**동작**:
- 기본: 빠른 plan 후 즉시 실행
- `--discuss`: 모호한 영역 먼저
- `--research`: 접근법 조사
- `--validate`: plan check + verify
- `--full`: 위 전부

**생성**: `.planning/quick/{번호}-{태스크}/PLAN.md`, `SUMMARY.md`

---

### 📊 페이즈 관리

| 명령 | 동작 |
|---|---|
| `/gsd-add-phase` | 로드맵 끝에 새 phase 추가 |
| `/gsd-insert-phase [N]` | 기존 phase 사이에 끼워 넣기 + 번호 재할당 |
| `/gsd-remove-phase [N]` | phase 제거 + 재번호 |
| `/gsd-plan-milestone-gaps` | audit가 발견한 미충족 요구사항 채울 phase 자동 생성 |

---

### 🎯 마일스톤 관리

| 명령 | 동작 |
|---|---|
| `/gsd-audit-milestone` | 현재 마일스톤이 definition of done 달성했는지 검증 |
| `/gsd-complete-milestone` | 모든 phase 완료 시 마일스톤 아카이브 + git tag |
| `/gsd-new-milestone [name]` | 다음 버전(v2 등) 시작 — 새 사이클 |

---

### 🌳 워크스트림 & 워크스페이스 (병렬 작업)

GSD는 **두 단계의 병렬성**을 지원합니다.

#### `/gsd-workstreams list|create|switch|complete`
**언제**: 한 프로젝트 안에서 여러 기능을 동시에 굴릴 때.

**동작**: 같은 코드베이스 안에 namespace 분리된 워크스트림 생성. 각 워크스트림이 자기 phase tracking. `switch`로 활성 전환.

**저장 위치**: `.planning/workstreams/{name}/`

#### `/gsd-new-workspace`
**언제**: 완전히 격리된 repo 카피로 병렬 작업할 때.

**동작**: worktree 또는 clone으로 별도 워크스페이스 생성. 각각 독립적인 GSD tracking.

| 명령 | 동작 |
|---|---|
| `/gsd-new-workspace` | 새 워크스페이스 생성 |
| `/gsd-list-workspaces` | 워크스페이스 목록 |
| `/gsd-remove-workspace` | 제거 |

#### 워크스트림 vs 워크스페이스

|  | Workstream | Workspace |
|---|---|---|
| 단위 | 한 repo 안의 namespace | 별도 repo 카피 (worktree/clone) |
| 무게 | 가벼움 | 무거움 (전체 repo) |
| 코드 충돌 | 같은 파일 동시 수정 가능성 있음 | 격리됨 |
| 머지 | 같은 repo니 자연스럽 | 별도 repo니 manual |
| 추천 | 같은 마일스톤의 병렬 phase | 다른 마일스톤 / 다른 실험 / 충돌 위험 큰 작업 |

---

### 🧪 리서치 & 실험

| 명령 | 언제 | 동작 |
|---|---|---|
| `/gsd-spike [idea] [--quick]` | 접근법 타당성 검증 | 2~5개 실험 with Given/When/Then 결과. `.planning/spikes/` |
| `/gsd-sketch [idea] [--quick]` | UI 디자인 변형 탐색 | 디자인 질문당 HTML 시안 2~3개. `.planning/sketches/` |
| `/gsd-spike-wrap-up` | spike 끝나고 결과 보존 | 발견을 프로젝트 로컬 스킬로 패키징 |
| `/gsd-sketch-wrap-up` | sketch 끝나고 결과 보존 | 디자인 결정을 스킬로 패키징 |

---

### 🔍 코드 품질 & 리뷰

| 명령 | 동작 |
|---|---|
| `/gsd-review` | 독립 리뷰 에이전트 dispatch — 크로스 AI 피어 리뷰 |
| `/gsd-secure-phase [N]` | 위협 모델 기반 보안 verify 적용 |
| `/gsd-ui-phase [N]` | 프론트엔드 phase 전 디자인 컨트랙트 (`{N}-UI-SPEC.md`) |
| `/gsd-ui-review [N]` | 구현된 프론트엔드의 6 pillar 시각 감사 |
| `/gsd-docs-update` | 코드와 문서 동기화 (doc-writer + doc-verifier) |

---

### 🧭 네비게이션 & 상태

| 명령 | 동작 |
|---|---|
| `/gsd-progress` | 현재 위치 / 완료 / 다음 작업 |
| `/gsd-help` | 전체 명령 레퍼런스 |
| `/gsd-update` | GSD 자체 업데이트 + changelog |
| `/gsd-manager` | 인터랙티브 phase 매니지먼트 |
| `/gsd-stats` | phase / plan / 요구사항 / git 메트릭 |
| `/gsd-milestone-summary [version]` | 종합 프로젝트 요약 |

---

### 🏗️ Brownfield (기존 코드베이스)

#### `/gsd-map-codebase [area]`
**언제**: 기존 프로젝트에 GSD 도입 전.

**동작**: 병렬 에이전트로 stack/architecture/conventions/concerns 분석 → 인덱스 생성. `/gsd-new-project` 가 이걸 보고 질문을 "추가할 것" 위주로 좁혀줌.

**생성**: `.planning/codebase-map/`

#### `/gsd-ingest-docs [dir]`
**언제**: 흩어진 PRD / ADR / 스펙 문서로 GSD 부트스트랩.

**동작**: 문서 분류 (PRD vs ADR vs spec) → `.planning/` 한 번에 합성, 충돌 해결 포함.

---

### 💾 세션 관리

| 명령 | 동작 |
|---|---|
| `/gsd-pause-work` | phase 도중 중단 — `HANDOFF.json` 에 상태 저장 |
| `/gsd-resume-work` | HANDOFF.json 에서 복구 |
| `/gsd-session-report` | 세션 종료 요약 (팀/본인용) |

---

### 💡 백로그 & 아이디어

| 명령 | 동작 |
|---|---|
| `/gsd-plant-seed <idea>` | 트리거 조건과 함께 forward 아이디어 — 조건 만족 시 자동 surface |
| `/gsd-add-backlog <desc>` | 999.x 번호로 활성 시퀀스 외 보관 |
| `/gsd-review-backlog` | 백로그 리뷰 → 활성 마일스톤 승격 또는 정리 |
| `/gsd-add-todo [desc]` | 빠른 todo 추가 |
| `/gsd-check-todos` | todo 리스트 |
| `/gsd-note <text>` | 마찰 없는 노트 — 나중에 todo 승격 가능 |

---

### 🐛 디버깅 & 진단

| 명령 | 동작 |
|---|---|
| `/gsd-debug [desc]` | 디버그 에이전트 dispatch. 세션 간 영속 상태 |
| `/gsd-forensics [desc]` | 실패한 워크플로우 사후 분석 — 멈춘 루프 / 누락 산출물 / git 이상 |
| `/gsd-health [--repair]` | `.planning/` 무결성 검증, `--repair` 자동 수정 |

---

### 🔧 유틸리티

| 명령 | 동작 |
|---|---|
| `/gsd-settings` | GSD 설정 에디터 (모델 / 워크플로우 토글 / git 전략) |
| `/gsd-set-profile <profile>` | 품질↔비용 트레이드오프 (`quality` / `balanced` / `budget` / `inherit`) |
| `/gsd-do <text>` | 자유 텍스트 → 적절한 명령 자동 라우팅 |
| `/gsd-profile-user [--questionnaire] [--refresh]` | 사용자 프로파일 분석 — GSD 응답 개인화 |
| `/gsd-join-discord` | 커뮤니티 |

---

## `.planning/` 디렉토리 구조

```
.planning/
├── config.json                # 모드 / granularity / 프로파일 / 워크플로우 토글
├── .gsd-manifest.json         # 설치 메타데이터 (full / minimal)
├── PROJECT.md                 # 프로젝트 비전 (항상 로드됨)
├── REQUIREMENTS.md            # v1 / v2 / out-of-scope
├── ROADMAP.md                 # phase 분해 ↔ 요구사항 매핑
├── STATE.md                   # 결정 / 블로커 / 위치 (세션 간 메모리)
│
├── 1-CONTEXT.md               # phase 1 디자인 선호 (discuss-phase)
├── 1-RESEARCH.md              # phase 1 도메인 리서치 (plan-phase)
├── 1-1-PLAN.md                # phase 1 의 atomic task 1
├── 1-2-PLAN.md                # atomic task 2
├── 1-1-SUMMARY.md             # task 1 산출 (execute-phase)
├── 1-VERIFICATION.md          # phase 검증
├── 1-UAT.md                   # 사용자 수락 테스트 (verify-work)
├── 1-UI-SPEC.md               # 디자인 컨트랙트 (ui-phase, optional)
│
├── research/                  # 초기 도메인 리서치 (new-project)
│   ├── architecture.md
│   ├── ecosystem.md
│   ├── features.md
│   └── pitfalls.md
├── codebase-map/              # 기존 코드 분석 (map-codebase)
├── quick/                     # ad-hoc 태스크 (gsd-quick)
│   └── 001-task/
├── spikes/                    # 실험 (spike)
├── sketches/                  # 디자인 시안 (sketch)
├── skills/                    # 프로젝트 로컬 스킬 (wrap-up)
├── threads/                   # 영속 컨텍스트 스레드
├── todos/ backlog/ seeds/     # 아이디어 큐
├── workstreams/               # 병렬 워크스트림
│   ├── main/
│   └── {name}/
├── HANDOFF.json               # pause/resume 상태
└── audit-{timestamp}.md       # 마일스톤 audit 결과
```

**원칙**:
- phase 산출물은 phase 번호 prefix로 평면 배치 (`1-CONTEXT.md`, `1-1-PLAN.md`, ...)
- SUMMARY / VERIFICATION 은 git history 에 영속
- `.planning/` 자체도 기본은 git 추적 (`planning.commit_docs` 토글)
- 워크스트림/quick/spike 는 메인 phase 시퀀스에서 분리됨

---

## 모델 프로파일

`/gsd-set-profile` 로 비용/품질 트레이드오프 선택.

| 프로파일 | 플래닝 | 실행 | 성격 |
|---|---|---|---|
| `quality` | Opus | Opus | 최고 품질, 비쌈 |
| `balanced` | Opus | Sonnet | 추천 기본값 |
| `budget` | Sonnet | Haiku | 저렴, 큰 양 |
| `inherit` | 현재 모델 | 현재 모델 | 단일 모델 |

---

## superpowers / gstack 와의 관계

| 도구 | 초점 | 강점 |
|---|---|---|
| [Superpowers](./superpowers.md) | 디시플린 (TDD/디버깅 절차) | 모든 작업에 일관된 방법론 |
| [gstack](./gstack.md) | 역할 (CEO/Eng/QA) | 풀-사이클 SaaS 개발 |
| GSD | 단계별 컨텍스트 관리 | 장기 / 큰 작업의 품질 유지 |

세 개 같이 쓰기: [DEV.to 가이드](https://dev.to/imaginex/a-claude-code-skills-stack-how-to-combine-superpowers-gstack-and-gsd-without-the-chaos-44b3)

---

## 변형 / 포팅

- [`gsd-build/gsd-2`](https://github.com/gsd-build/gsd-2) — Pi SDK 기반 단독 CLI. Claude Code 없이도 동작
- [`rokicool/gsd-opencode`](https://github.com/rokicool/gsd-opencode) — OpenCode 포팅
- [`itsjwill/gsd-pro`](https://github.com/itsjwill/gsd-pro) — 멀티 모델 라우팅 + 롤백/복구 추가 fork
- [`jnuyens/gsd-plugin`](https://github.com/jnuyens/gsd-plugin) — 성능 최적화 plugin 패키징, 81 커맨드 + 21 에이전트

---

## 라이선스

MIT.

---

## 참고

- 메인: https://github.com/gsd-build/get-shit-done
- USER GUIDE: https://github.com/gsd-build/get-shit-done/blob/main/docs/USER-GUIDE.md
- 인터랙티브 레슨: https://ccforeveryone.com/gsd
- 워크플로우 분석: https://www.codecentric.de/en/knowledge-hub/blog/the-anatomy-of-claude-code-workflows-turning-slash-commands-into-an-ai-development-system
- 핸즈온 워크샵: [`../../workshops/extensions/gsd-workshop.md`](../../workshops/extensions/gsd-workshop.md)
