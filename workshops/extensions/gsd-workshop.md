# GSD (Get Shit Done) 핸즈온 워크샵

[GSD](../../docs/extensions/gsd.md)는 Claude Code 위에 얹는 **스펙 드리븐 개발 시스템**입니다. 큰 작업도 **단계별 마크다운 산출물**로 쪼개서, 매 단계가 신선한 컨텍스트에서 시작되도록 강제합니다.

이 워크샵에서는 **간단한 CLI 할 일 앱**을 GSD로 만들어보면서:
- 각 슬래시 커맨드가 실제로 무엇을 하는지
- `.planning/` 디렉토리에 어떤 파일이 쌓이는지
- 워크스페이스/워크스트림 같은 병렬 작업을 어떻게 굴리는지

확인합니다. 매 단계마다 **"결과 확인" 체크포인트**가 있으니 따라가면서 직접 눈으로 보세요.

> 사전 학습: [Claude Code 워크샵](../agents/claude-code-workshop.md) → 본 워크샵
> 레퍼런스: [`docs/extensions/gsd.md`](../../docs/extensions/gsd.md)

---

## 0. 사전 준비

| 항목 | 확인 명령 |
|---|---|
| Claude Code 2.1.88+ | `claude --version` |
| Node.js 18+ | `node --version` |
| Anthropic 인증 | `claude` 실행 → 로그인 상태 |
| Git | `git --version` |

없으면:
- Claude Code: [`claude-code-workshop.md`](../agents/claude-code-workshop.md) 의 1~3절
- Node: [`../../docs/nodejs.md`](../../docs/nodejs.md)

---

## 1. 빈 프로젝트 폴더 준비

```bash
mkdir ~/Desktop/gsd-demo
cd ~/Desktop/gsd-demo
git init
```

`.gitignore` 한 줄:

```bash
echo "node_modules" > .gitignore
git add .gitignore
git commit -m "chore: init"
```

### ✅ 결과 확인

```bash
ls -la
# .git/ 와 .gitignore 만 있어야 함

git log --oneline
# 첫 커밋 1개
```

---

## 2. GSD 설치

이 폴더 안에서 (글로벌 아닌 프로젝트 단위 추천 — 첫 시도라면):

```bash
npx get-shit-done-cc@latest
```

설치 마법사가 뜨고:
- 어디 설치할지 (Project / Global)
- `--minimal` vs full
- 모델 프로파일 (`quality` / `balanced` / `budget`)

처음이라면 **Project / full / balanced** 추천.

### ✅ 결과 확인

```bash
ls .claude/
# commands/ skills/ agents/ 같은 게 생겨 있어야 함

ls .claude/commands | head -20
# gsd-new-project.md, gsd-discuss-phase.md, ... 가 보여야 함

cat .planning/.gsd-manifest.json 2>/dev/null
# 설치 메타데이터 (없을 수도 — 첫 명령 실행 시 생성)
```

Claude Code 재시작:

```bash
claude
```

세션 안에서:

```
> /gsd-help
```

명령어 목록이 길게 출력되면 설치 성공.

---

## 3. 새 프로젝트 초기화 — `/gsd-new-project`

이 워크샵의 목표 앱: **TypeScript로 만든 간단한 CLI 할 일 앱** (add / list / done / remove, JSON 파일에 저장).

```
> /gsd-new-project
```

GSD가 인터뷰를 시작합니다. 예시 답변:

| 질문 (의역) | 예시 답변 |
|---|---|
| 무엇을 만드시나요? | TypeScript CLI 할 일 앱 |
| 사용자는? | 본인, 터미널에서 |
| 데이터 저장은? | 로컬 JSON 파일 (`~/.gsd-demo-todos.json`) |
| 명령은? | add, list, done, remove |
| 우선순위? | v1: 4개 명령 + 파일 저장. v2: 카테고리 / 정렬 옵션 |

GSD는 응답을 받으면 병렬 리서치 에이전트를 돌립니다 (도메인 조사 — CLI 라이브러리 비교, JSON 저장 best practice 등). 끝나면 로드맵을 보여주고 승인 요청.

### ✅ 결과 확인

```bash
ls .planning/
# PROJECT.md  REQUIREMENTS.md  ROADMAP.md  STATE.md  research/  config.json

cat .planning/PROJECT.md
# 프로젝트 비전 (당신이 답한 내용 + 정리)

cat .planning/REQUIREMENTS.md
# v1 / v2 / out-of-scope 분리됨

cat .planning/ROADMAP.md
# 보통 phase 2~4개 정도. 이 예제라면:
#   Phase 1: 코어 데이터 모델 + JSON 저장
#   Phase 2: add/list 명령
#   Phase 3: done/remove 명령
#   Phase 4: 폴리시 (에러 메시지, 도움말)

ls .planning/research/
# architecture.md  ecosystem.md  features.md  pitfalls.md
```

다음 단계 가기 전 ROADMAP.md 한 번 읽어보세요. 마음에 안 들면:

```
> phase 2와 3 합쳐줘. 너무 잘게 쪼개졌어.
```

GSD가 ROADMAP 수정.

---

## 4. Phase 1 의 디자인 결정 — `/gsd-discuss-phase`

```
> /gsd-discuss-phase 1
```

GSD가 phase 1의 **모호한 영역**을 식별해서 질문합니다. 예:
- JSON 파일 위치 (`~/.gsd-demo-todos.json` vs 프로젝트 로컬)?
- ID 형식 (incremental int / uuid)?
- 빈 파일 / 손상 파일 처리?

답변하면 결정사항을 컨텍스트 문서로 박아 둡니다.

### ✅ 결과 확인

```bash
cat .planning/1-CONTEXT.md
```

이 문서에 다음 단계가 따라야 할 결정이 모두 적혀 있어야 합니다. 모호함 없이.

> 💡 팁: `--analyze` 플래그 붙이면 트레이드오프 분석도 추가됨. `/gsd-discuss-phase 1 --analyze`

---

## 5. 실행 가능한 plan 만들기 — `/gsd-plan-phase`

```
> /gsd-plan-phase 1
```

GSD 가:
1. CONTEXT.md 읽고 **추가 리서치** (어떤 라이브러리 쓸지, 코드 구조 등)
2. **2~3개 atomic task plan** 생성. 각 plan은 fresh context에서 단독 실행 가능
3. plan-checker 가 검증 → 통과 못 하면 자동 루프로 재작성

### ✅ 결과 확인

```bash
ls .planning/
# 1-RESEARCH.md, 1-1-PLAN.md, 1-2-PLAN.md ... 가 추가되어야 함

cat .planning/1-RESEARCH.md
# 라이브러리 비교 / 권장 접근법 등

cat .planning/1-1-PLAN.md
# XML 구조의 task plan:
#   <objective>...</objective>
#   <files-to-create>...</files-to-create>
#   <verify>...</verify>
```

plan을 보고 마음에 안 들면 `/gsd-plan-phase 1` 다시 실행 또는 GSD에게 직접 수정 지시.

> 💡 팁: 이 시점에 plan을 한 번 꼼꼼히 보고 수정하는 게 가장 ROI가 큽니다. 잘못된 plan은 잘못된 코드를 양산.

---

## 6. 실제 코드 만들기 — `/gsd-execute-phase`

```
> /gsd-execute-phase 1
```

GSD 가:
1. plan들을 dependency 분석 → wave 그룹
2. 독립 plan은 **병렬** 실행 (각각 fresh 200K 컨텍스트)
3. 완료된 task마다 **atomic git commit**
4. phase 끝나면 검증

### ✅ 결과 확인

```bash
# 실제 파일들이 생겼는지
ls src/   # 또는 프로젝트 구조에 따라
cat package.json

# git history (각 task가 별도 commit)
git log --oneline
# feat(phase-1.1): ...
# feat(phase-1.2): ...

# planning 산출물
ls .planning/
# 1-1-SUMMARY.md, 1-2-SUMMARY.md, 1-VERIFICATION.md 가 추가됨

cat .planning/1-VERIFICATION.md
# phase 1 의 약속이 실제 코드와 일치하는지 자동 검증 결과
```

### 빌드 / 실행해보기

```bash
# 보통 GSD가 setup도 같이 끝내놓음
pnpm install   # 또는 npm i

# CLI 실행
node dist/cli.js list   # 정확한 진입점은 ROADMAP/PLAN 따라
```

이 단계에서 **에러가 나면 바로 다음 verify 단계로**.

---

## 7. 사용자 수락 테스트 — `/gsd-verify-work`

```
> /gsd-verify-work 1
```

GSD 가 phase 1 산출에서 testable deliverable을 추출하고 하나씩 묻습니다:

```
✅ 빈 JSON 파일에서 시작해서 add 가능한가요? (y/n)
✅ list가 빈 상태에서 적절한 메시지를 보여주나요? (y/n)
...
```

**y/n** 으로 답. 실패한 게 있으면 GSD가 debug 에이전트를 dispatch — 근본 원인 진단 + fix plan 생성.

### ✅ 결과 확인

```bash
cat .planning/1-UAT.md
# UAT 결과 + (실패 있었다면) fix plan
```

실패가 있어 fix plan이 생겼다면:

```
> /gsd-execute-phase 1
```

다시 한 번 실행해서 fix plan을 처리. 이 사이클이 작은 만큼 큰 회귀 부담 없이 다듬을 수 있는 게 장점.

---

## 8. PR 만들기 — `/gsd-ship`

git 원격이 없으면 잠시 만들어 둡니다:

```bash
gh repo create gsd-demo --private --source=. --remote=origin --push
```

(GitHub CLI 인증 필요. [github-workshop.md](../collaboration/github-workshop.md) 참고)

```
> /gsd-ship 1
```

GSD 가:
1. PR body 자동 생성 (변경사항 요약, 테스트 노트)
2. `.planning/` 제외 필터로 PR 만들기
3. 원격 PR 생성

### ✅ 결과 확인

```bash
gh pr view --web
# 브라우저에서 자동 생성된 PR 확인

# 또는 터미널에서
gh pr view
```

PR 내용:
- 어떤 phase 가 들어갔는지
- 추가/변경된 파일 요약
- 어떤 테스트를 어떻게 돌리는지

> 💡 처음에는 `--draft` 로 만들어 두고 직접 다듬는 걸 추천: `/gsd-ship 1 --draft`

---

## 9. 다음 phase 진행 — `/gsd-next`

여기서부터는 **명령 자동 라우팅** 활용:

```
> /gsd-next
```

GSD 가 현재 상태(Phase 1 ship됨, Phase 2 미시작)를 감지하고 **`/gsd-discuss-phase 2`** 를 자동 실행. 한 번씩 누르며 진행하면 됩니다.

또는 한 방에 진행:

```
> /gsd-discuss-phase 2 --chain
```

discuss → plan → execute 까지 자동.

### ✅ 결과 확인 (각 phase 마다)

```bash
# 매 phase 끝날 때마다
ls .planning/ | grep -E "^[0-9]"
# 1-CONTEXT.md  1-RESEARCH.md  1-1-PLAN.md  1-VERIFICATION.md
# 2-CONTEXT.md  2-RESEARCH.md  ...

git log --oneline | head
# 각 task가 별도 커밋

cat .planning/STATE.md
# 현재 위치와 결정사항이 누적
```

---

## 10. 작은 작업은 `/gsd-fast` 또는 `/gsd-quick`

phase로 만들 정도가 아닌 일은 가벼운 모드 사용.

### 즉시 실행

```
> /gsd-fast README에 한국어 설치 가이드 추가
```

플래닝 없이 바로 작업. atomic commit 보장.

### 작은 plan + 실행

```
> /gsd-quick CLI 도움말 메시지 깔끔하게 다듬어줘
```

`.planning/quick/` 안에 가벼운 plan + summary 남김.

### ✅ 결과 확인

```bash
ls .planning/quick/
# 001-readme-add-korean/
# 002-cli-help-cleanup/

cat .planning/quick/002-cli-help-cleanup/SUMMARY.md
```

---

## 11. 실험 — `/gsd-spike` & `/gsd-sketch`

### Spike: "이거 가능한가?" 검증

```
> /gsd-spike 할 일을 자연어로 받고 LLM으로 파싱하는 거 가능한가?
```

GSD 가 2~5개 작은 실험을 돌리고 **Given/When/Then 결과**를 남깁니다.

### Sketch: UI 시안 탐색

(이 워크샵 앱은 CLI라 less 적용. 웹 UI 프로젝트라면)

```
> /gsd-sketch 할 일 추가 폼의 layout 변형 3개
```

→ `.planning/sketches/` 에 인터랙티브 HTML 시안 생성.

### ✅ 결과 확인

```bash
ls .planning/spikes/
cat .planning/spikes/{idea}/verdicts.md
```

좋은 실험은 `/gsd-spike-wrap-up` 으로 프로젝트 로컬 스킬로 박제.

---

## 12. 병렬 작업 — Workstream vs Workspace

큰 프로젝트에선 두 종류의 병렬성을 알아두면 유용합니다.

### 워크스트림: 같은 repo, 다른 phase 줄기

`refactor/auth` 와 `feat/payment` 를 동시에 굴리고 싶을 때.

```
> /gsd-workstreams create refactor-auth
> /gsd-workstreams create feat-payment
> /gsd-workstreams switch refactor-auth
> /gsd-new-project
   (이 워크스트림만의 ROADMAP 작성)
```

### ✅ 확인

```bash
ls .planning/workstreams/
# main/  refactor-auth/  feat-payment/

cat .planning/workstreams/refactor-auth/ROADMAP.md
# 이 줄기만의 phase 시퀀스
```

전환:

```
> /gsd-workstreams switch feat-payment
```

머지 친화적이지만 **같은 파일을 동시에 만지면 충돌** 가능.

### 워크스페이스: 격리된 repo 카피

`feat/payment` 가 데이터베이스 스키마를 크게 바꾸고 메인 작업과 충돌 가능성이 높을 때.

```
> /gsd-new-workspace
   (워크스페이스 이름: payment-experiment)
```

GSD 가 worktree 또는 clone 만들고 거기서 별도 GSD 트래킹.

### ✅ 확인

```bash
> /gsd-list-workspaces
# main          (현재 폴더)
# payment-experiment  (../gsd-demo-payment-experiment 등)
```

각 워크스페이스에서 `/gsd-new-project` → 완전히 독립적 사이클.

| 상황 | 추천 |
|---|---|
| 같은 마일스톤의 여러 기능 분기 | Workstream |
| 큰 실험 / 위험한 리팩터 / 다른 마일스톤 | Workspace |

---

## 13. 마일스톤 / audit / 히스토리

### 마일스톤 audit

phase 다 끝났다고 생각하면:

```
> /gsd-audit-milestone
```

GSD 가 정의된 v1 요구사항을 다 충족했는지 검증. 빠진 게 있으면:

```
> /gsd-plan-milestone-gaps
```

자동으로 gap을 메울 phase를 추가.

### 마일스톤 마무리

다 끝났다면:

```
> /gsd-complete-milestone
```

git tag (`v1.0.0` 같은) + 마일스톤 아카이브.

### v2 시작

```
> /gsd-new-milestone v2
```

새 사이클 (질문 → 리서치 → 로드맵). v1 산출은 `STATE.md` 에 컨텍스트로 남음.

### ✅ 확인

```bash
git tag
# v1.0.0

ls .planning/audit-*.md
# audit-2026-04-27T15-30-00.md

cat .planning/STATE.md
# 마일스톤 히스토리 + 현재 위치
```

---

## 14. 세션 관리 (중간에 끊을 때)

진행 중이던 phase 가 있는데 잠시 끊어야 한다면:

```
> /gsd-pause-work
```

→ `.planning/HANDOFF.json` 생성. 실행 상태 (어느 wave 어느 task) 저장됨.

다음 세션:

```
> /gsd-resume-work
```

→ HANDOFF.json 에서 복구 + 이어서 실행.

### ✅ 확인

```bash
cat .planning/HANDOFF.json
# 일시정지 시점의 모든 상태
```

---

## 15. 트러블슈팅

### `.planning/` 이 망가졌다 / 일관성 깨짐

```
> /gsd-health
> /gsd-health --repair
```

자동 진단 + 수정.

### 워크플로우가 멈췄다 / 이상한 상태

```
> /gsd-forensics 갑자기 plan-phase가 무한 루프
```

사후 분석 — 멈춘 루프 / 누락 산출물 / git 이상 탐지.

### 토큰 비용이 너무 큼

```
> /gsd-set-profile budget
```

또는 처음부터 `--minimal` 로 설치.

### plan이 자꾸 잘못 만들어짐

CONTEXT.md 가 너무 모호한 경우. `/gsd-discuss-phase N --analyze` 로 다시 깊게.

### Claude 가 GSD 명령을 못 알아본다

- Claude Code 2.1.88+ 인지 확인 (`claude --version`)
- 재시작 (`/exit` 후 다시 `claude`)
- `.claude/commands/` 에 gsd-*.md 파일 있는지

---

## 16. 전체 흐름 요약

```
[사전 준비]
    ↓
git init + npx get-shit-done-cc@latest
    ↓
/gsd-new-project          → PROJECT / REQUIREMENTS / ROADMAP
    ↓
/gsd-discuss-phase 1      → 1-CONTEXT.md
    ↓
/gsd-plan-phase 1         → 1-RESEARCH + 1-N-PLAN
    ↓
/gsd-execute-phase 1      → 코드 + atomic commits + 1-VERIFICATION
    ↓
/gsd-verify-work 1        → 1-UAT (실패 시 fix plan → 다시 execute)
    ↓
/gsd-ship 1               → PR
    ↓
/gsd-next                 → phase 2부터 자동 라우팅
    ↓
[모든 phase 완료]
    ↓
/gsd-audit-milestone      → 미충족 발견 시 /gsd-plan-milestone-gaps
    ↓
/gsd-complete-milestone   → git tag
    ↓
/gsd-new-milestone v2     → 다음 사이클 🎉
```

수고하셨습니다 🎉

---

## 17. 더 깊이 보기

- [GSD 레퍼런스](../../docs/extensions/gsd.md) — 70+ 명령 전체 정리
- [공식 USER GUIDE](https://github.com/gsd-build/get-shit-done/blob/main/docs/USER-GUIDE.md)
- [Claude Code Skills Stack: superpowers + gstack + GSD 함께 쓰기](https://dev.to/imaginex/a-claude-code-skills-stack-how-to-combine-superpowers-gstack-and-gsd-without-the-chaos-44b3)
- 인터랙티브 레슨: https://ccforeveryone.com/gsd
