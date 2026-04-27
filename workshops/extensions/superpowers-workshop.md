# Superpowers 핸즈온 워크샵

[Superpowers](../../docs/extensions/superpowers.md) (obra/superpowers, v5.0.7+) 는 Claude Code 위에 **소프트웨어 개발 방법론을 디시플린으로 강제**하는 스킬 프레임워크입니다.

핵심 워크플로우는 **3단계**:

```
Brainstorming → Planning → Execution
   ↓                ↓           ↓
 요구사항 합의    상세 계획    fresh 서브에이전트가 task 단위로
                              (2~5분 단위, 검증 단계 포함)
```

이 워크샵에서는 **Markdown 노트를 모아 정적 사이트로 빌드하는 작은 CLI** 를 만들어보면서:
- 어떻게 `/brainstorm` 이 멍한 요구사항을 구체화하는지
- `/write-plan` 이 어떤 plan 파일을 만드는지
- `/execute-plan` 이 어떻게 서브에이전트로 분할 실행하는지
- `systematic-debugging`, `verification-before-completion` 같은 메타 스킬이 자동 발동하는 모습

을 직접 보고 갑니다.

> 사전 학습: [Claude Code 워크샵](../agents/claude-code-workshop.md)
> 레퍼런스: [`docs/extensions/superpowers.md`](../../docs/extensions/superpowers.md)

---

## 0. 사전 준비

| 항목 | 확인 |
|---|---|
| Claude Code (최신) | `claude --version` |
| Anthropic 인증 | `claude` 실행 → `/login` 상태 |
| Git | `git --version` |

---

## 1. 설치 — Plugin 마켓플레이스

Claude Code 안에서 한 줄.

```
> /plugin install superpowers@claude-plugins-official
```

대안 (커뮤니티 마켓플레이스 — 최신 dev 버전):

```
> /plugin marketplace add obra/superpowers-marketplace
> /plugin install superpowers@superpowers-marketplace
```

설치 후 Claude Code 재시작 (한 번 `/exit` 후 다시 `claude`).

### ✅ 결과 확인

```
> /plugin
```

목록에 `superpowers v5.0.7+` (Core skills library for Claude Code) 가 보이면 OK.

세션 시작 시 자동으로 `using-superpowers` 메타스킬이 발동되어 다음 같은 흐름이 보입니다:

```
[using-superpowers 스킬 활성화]
이 세션에선 다음 스킬들이 사용 가능합니다:
- brainstorming
- writing-plans
- executing-plans
- test-driven-development
- systematic-debugging
- verification-before-completion
- using-git-worktrees
- dispatching-parallel-agents
- writing-skills
- requesting-code-review
- receiving-code-review
- subagent-driven-development
- finishing-a-development-branch
```

스킬 본체는 다음 위치에 깔립니다:

```bash
ls ~/.claude/plugins/superpowers/skills/
# brainstorming/   debugging/   tdd/   ...
```

각 스킬은 `SKILL.md` 한 파일이 핵심.

---

## 2. 빈 프로젝트 폴더

```bash
mkdir ~/Desktop/sp-demo
cd ~/Desktop/sp-demo
git init
echo "node_modules" > .gitignore
git add .gitignore
git commit -m "chore: init"
claude
```

---

## 3. Phase 1: Brainstorming — `/brainstorm`

**중요**: Superpowers는 "구현 시작 전 brainstorm" 을 강제합니다. 그냥 "X 만들어줘" 라고 던지면 자동으로 brainstorming 스킬이 발동되어 질문이 시작됩니다.

```
> 마크다운 노트들을 정적 사이트로 빌드하는 CLI 만들어줘
```

또는 명시적으로:

```
> /brainstorm 마크다운 노트들을 정적 사이트로 빌드하는 CLI
```

Superpowers가 다음 같은 인터뷰를 시작합니다 (실제 질문은 매번 다름):

| 영역 | 예시 질문 |
|---|---|
| 입력 | 마크다운 파일은 어디에서 읽나? 디렉토리 구조 정해진 게 있나? |
| 출력 | HTML만? 또는 RSS / sitemap.xml 도? |
| 스타일 | 테마 시스템? 단일 CSS? 사용자 정의? |
| 빌드 | watch 모드? 점진 빌드? 캐시? |
| 배포 | `dist/` 만? GitHub Pages 같은 데 푸시까지? |

**핵심**: 답변을 작은 청크로 나눠 받습니다. 한 번에 다 안 묻고 다음 단계 가기 전 **사용자 승인**.

### ✅ 결과 확인

브레인스토밍이 끝나면 보통:

```
> 좋아 다 정리됐어. 이제 plan 만들자.
```

또는 Claude가 알아서 "specification 정리해뒀고, plan 만들어도 될까요?" 라고 묻습니다.

대화 안에서 만들어진 spec/요약을 한 번 읽어보고:

```
> 지금까지 합의된 spec 요약해줘
```

스펙 한 화면 분량으로 다시 한 번 검토 → 마음에 안 드는 곳 짚고 수정.

> 💡 brainstorming 의 가장 큰 효과: "내가 뭘 원하는지" 사용자도 모르고 시작했던 게 끝나면 명확해짐. 이게 plan 품질을 결정합니다.

---

## 4. Phase 2: Planning — `/write-plan`

```
> /write-plan
```

Superpowers가 brainstorm 결과를 바탕으로 **구조화된 implementation plan** 을 작성합니다.

각 task는 다음 속성:
- 정확한 파일 경로 (`src/builder.ts:1-80`)
- 2~5분 단위 (서브에이전트가 fresh context에서 처리 가능한 크기)
- 검증 단계 (어떻게 "끝났는지" 알지)
- 의존성 (이 task가 다른 task에 의존하는지)

### ✅ 결과 확인

plan은 보통 마크다운 파일로 저장됩니다. 위치는 프로젝트 단위로 다를 수 있는데 일반적으로:

```bash
ls -la
# .plan.md 또는 plans/ 같은 디렉토리

cat .plan.md  # 또는 plans/<feature>.md
```

plan 내용 예 (요약):

```markdown
# Plan: Markdown to Static Site Builder

## Phase 1: 코어 빌더
- [ ] Task 1.1: 디렉토리 트래버스 + 마크다운 파일 수집 (src/scanner.ts, ~3분)
- [ ] Task 1.2: marked 로 HTML 변환 + frontmatter 파싱 (src/render.ts, ~5분)
- [ ] Task 1.3: dist/ 출력 + 디렉토리 미러링 (src/writer.ts, ~3분)
- 검증: `examples/` 디렉토리로 테스트, dist/ 결과 수동 확인

## Phase 2: 인덱스 / 메타
- [ ] Task 2.1: index.html 자동 생성 (~5분)
- [ ] Task 2.2: sitemap.xml (~3분)
- 검증: 모든 노트가 인덱스에 링크되는지

## Phase 3: CLI 인터페이스
- [ ] Task 3.1: commander 로 build / watch 명령 (~5분)
- ...
```

**여기서 직접 plan을 손으로 수정하는 것이 권장됩니다.** 모호한 게 있거나 task 단위가 너무 크면 잘라달라고 하세요:

```
> Task 1.2 너무 큰 거 같아. 마크다운 변환과 frontmatter를 분리해서 task 두 개로 나눠줘.
```

> 💡 plan 단계에서 1분 더 쓰는 게 execute 단계에서 30분 절약합니다.

---

## 5. Phase 3: Execution — `/execute-plan`

```
> /execute-plan
```

Superpowers가 plan의 task를 **fresh 서브에이전트** 에 분할 dispatch:

```
[wave 1] Task 1.1 → subagent A (fresh 200K)
                    완료 → spec 검증 → 코드 품질 검증 → commit
[wave 1] Task 1.2 → subagent B (fresh 200K)  (병렬)
[wave 2] Task 2.1 → ...                       (Phase 1 완료 후)
```

각 task에 대해 **2 stage 리뷰**:
1. **Spec compliance** — plan에 적힌 요구사항을 충족했는가
2. **Code quality** — 네이밍, 구조, 엣지 케이스 처리

문제가 있으면 자동으로 수정 / 또는 사용자 확인 요청.

### ✅ 결과 확인

진행 상태:

```bash
# git history (각 task 완료 시 commit)
git log --oneline
# feat: scanner: walk markdown files
# feat: renderer: parse frontmatter and convert MD->HTML
# feat: writer: emit dist/ tree
# ...

# 실제 파일들
ls src/
# scanner.ts  render.ts  writer.ts  cli.ts

# plan 진행률 — 보통 plan 파일의 [ ] 가 [x] 로 바뀜
grep -c "\[x\]" .plan.md
grep -c "\[ \]" .plan.md
```

빌드 시도:

```bash
pnpm install
pnpm build
node dist/cli.js build examples/
ls dist/
```

---

## 6. 실수가 발견됐을 때 — `systematic-debugging` 자동 발동

빌드 결과가 이상하다면:

```
> dist/index.html 이 빈 파일이야. 왜 그래?
```

Superpowers가 **버그 발견 신호** 를 감지하고 자동으로 `systematic-debugging` 스킬을 발동합니다.

이 스킬은 4단계:

1. **재현 가능한 최소 케이스 만들기** — 한 줄 명령으로 재현되도록
2. **가설 3개 이상** 적기 — 잘못된 경로? 빈 입력? 인코딩?
3. **각 가설 검증 방법** 결정
4. **검증 → 가설 좁히기 → 근본 원인** 확정

YOLO 패치 (그냥 if문 추가) 를 명시적으로 거부합니다.

### ✅ 결과 확인

debugging 절차가 진행되는 동안 화면에 다음 같은 출력이 보입니다:

```
[systematic-debugging 활성]

가설 1: scanner가 빈 디렉토리만 봄 → 검증: scanner.ts 단독 테스트
가설 2: render가 frontmatter 파싱 실패 → 검증: render.ts에 console.log
가설 3: writer가 stream을 닫지 않음 → 검증: ...
```

근본 원인 확정 후 fix → atomic commit. **방치된 디버그 출력이나 임시 코드** 도 같이 정리됨.

---

## 7. 완료 직전 — `verification-before-completion` 강제 발동

작업이 끝났다고 주장하기 직전:

```
> 다 했어. 이제 commit 하자
```

Superpowers가 자동으로 `verification-before-completion` 스킬 발동. **검증 없이 "끝났다" 라고 못 합니다**.

이 스킬은:
- 실제로 빌드 명령 실행 — `pnpm build` 통과?
- 테스트 실행 — `pnpm test`
- 에지 케이스 확인 — 빈 입력? 잘못된 마크다운?
- 새로 만든 파일 / 함수 모두 사용되는지 (dead code 검출)

### ✅ 결과 확인

화면에 나오는 검증 출력을 그대로 보세요:

```
[verification-before-completion]

▶ pnpm build
✅ 빌드 성공

▶ pnpm test
✅ 12 passed, 0 failed

▶ 빈 examples/ 로 테스트
✅ 적절한 에러 메시지 출력

▶ dead code 스캔
⚠️ src/utils/dateformat.ts 가 어디서도 import 되지 않음
   → 제거하거나 사용처 추가 필요
```

마지막 단계의 **경고**도 무시 안 됨 — 처리해야 진행.

---

## 8. PR 만들기 — `requesting-code-review` & `finishing-a-development-branch`

검증 통과하면:

```
> 이제 PR 만들자
```

먼저 자동 발동되는 스킬들:

### `requesting-code-review`
서브에이전트(`code-reviewer`) 를 dispatch 해서 **독립적인 리뷰**.
- plan과 실제 구현이 일치하나
- 안전성 / 성능 회귀
- 컨벤션 어긴 부분

리뷰 결과를 사용자에게 보여주고 수정할지 물음.

### `receiving-code-review`
리뷰가 들어왔을 때 **performative 동의 금지**. "네 알겠습니다 고치겠습니다" 가 아니라:
- 기술적으로 검증
- 동의 안 하면 이유 적기
- 동의하면 정확히 어디를 어떻게 고칠지

### `finishing-a-development-branch`
머지 / PR / cleanup 옵션을 구조화해서 제시.

### ✅ 결과 확인

```bash
gh pr view --web
```

생성된 PR에 다음이 들어 있어야 합니다:
- 변경사항 요약
- 검증 결과 (build/test/edge case)
- (있다면) 리뷰에서 짚인 사항과 대응

---

## 9. 큰 작업은 worktree 로 — `using-git-worktrees`

다음 큰 기능을 시작하려는데 현재 작업이 깨끗하지 않다면:

```
> 다음 기능 시작하기 전에 격리된 환경으로 가자
```

`using-git-worktrees` 가 발동:

```bash
# 자동으로 실행됨
git worktree add ../sp-demo-feat-themes -b feat/themes
```

새 디렉토리에서 작업하면 메인 폴더와 충돌 없음.

### ✅ 결과 확인

```bash
git worktree list
# /Users/.../sp-demo            <commit>  [main]
# /Users/.../sp-demo-feat-themes <commit> [feat/themes]
```

병렬 작업 끝나면 normal merge / PR.

---

## 10. 독립 작업 2개 이상 — `dispatching-parallel-agents`

```
> README 한국어판도 만들고, examples/ 디렉토리에 샘플 노트 5개도 추가해줘
```

두 작업이 독립이면 자동으로 `dispatching-parallel-agents` 발동:

```
[parallel] Agent 1: README ko 번역
[parallel] Agent 2: examples/ 샘플 작성
```

서로 영향 없이 두 결과를 동시에 받아옴.

### ✅ 결과 확인

```bash
git log --oneline -5
# docs(ko): README 한국어판
# docs: examples 샘플 노트 추가
```

두 commit 이 거의 동시에 찍힘.

---

## 11. 커스텀 스킬 만들기 — `writing-skills`

자기 팀/프로젝트 만의 디시플린을 스킬화:

```
> /skill-creator
```

또는:

```
> writing-skills 사용해서 우리 팀 PR 체크리스트 스킬 만들어줘
```

`writing-skills` 가 발동, 인터뷰 후 SKILL.md 작성.

### ✅ 결과 확인

```bash
ls ~/.claude/skills/
# our-pr-checklist/

cat ~/.claude/skills/our-pr-checklist/SKILL.md
```

```markdown
---
name: our-pr-checklist
description: Use before creating any PR — enforces our team's quality gates
---

PR 만들기 전 다음 모두 확인:
1. `pnpm test` 통과
2. CHANGELOG 업데이트
3. 마이그레이션 있으면 staging dry-run
4. 한국어 PR 제목 + 영어 본문
```

다음 세션부터 PR 만들 때 자동 호출.

---

## 12. 트러블슈팅

### 스킬이 자동 발동 안 함

`description` 이 모호하면 Claude가 호출 시점을 못 잡습니다.

```bash
# 스킬 description 확인
grep "^description:" ~/.claude/plugins/superpowers/skills/*/SKILL.md
```

description 은 "Use when ..." 으로 시작해서 발동 신호를 명확히 해야 함.

### 너무 많이 발동되어 거추장스러움

작은 작업에 brainstorming 까지 강제되면 답답함. 명시적으로 우회:

```
> 이건 한 줄 fix야. brainstorming 건너뛰고 바로 수정.
```

또는 CLAUDE.md 에 적기:

```markdown
- 한 줄 변경 / typo fix / 의존성 업데이트는 brainstorming 생략
```

### plan / execute 중 멈춤

```
> /clear
> /execute-plan resume
```

대부분의 스킬은 진행 상태를 plan 파일에 [x] 로 박아두므로 resume 가능.

### 마켓플레이스 인증 실패

```
> /plugin marketplace remove claude-plugins-official
> /plugin marketplace add anthropic/plugin-marketplace
> /plugin install superpowers@claude-plugins-official
```

---

## 13. 전체 흐름 요약

```
[Plugin install]
   /plugin install superpowers@claude-plugins-official
   ↓
[새 작업 요청]
   "X 만들어줘"
   ↓
brainstorming        → 요구사항 합의 (작은 청크 + 승인)
   ↓
writing-plans        → .plan.md (task 2~5분 단위)
   ↓
executing-plans      → fresh 서브에이전트 분할 + 2 stage 리뷰
   ↓
(중간) systematic-debugging  → 버그 발생 시 자동
(중간) verification-before-completion → "끝났다" 주장 직전
   ↓
requesting-code-review → 독립 리뷰
   ↓
finishing-a-development-branch → PR / merge
```

**병렬 / 격리:**
- using-git-worktrees → 격리된 작업
- dispatching-parallel-agents → 독립 작업 2개+

**커스터마이징:**
- writing-skills → 우리 팀 디시플린 스킬화

---

## 14. 다른 스택과 함께 쓰기

| 조합 | 효과 |
|---|---|
| Superpowers + [GSD](./gsd-workshop.md) | 디시플린 (Superpowers) + 단계별 컨텍스트 (GSD). 큰 프로젝트 추천. |
| Superpowers + [gstack](./gstack-workshop.md) | 디시플린 (Superpowers) + 역할 (gstack). SaaS 풀사이클 추천. |
| Superpowers 단독 | 1인 / 소규모 / 학습 단계 |

같이 쓰는 가이드: [DEV.to: Superpowers + gstack + GSD](https://dev.to/imaginex/a-claude-code-skills-stack-how-to-combine-superpowers-gstack-and-gsd-without-the-chaos-44b3)

---

## 15. 더 보기

- 레퍼런스: [`docs/extensions/superpowers.md`](../../docs/extensions/superpowers.md)
- 메인 repo: https://github.com/obra/superpowers
- Lab (실험적 스킬): https://github.com/obra/superpowers-lab
- Marketplace: https://github.com/obra/superpowers-marketplace
- 공식 anthropic 페이지: https://claude.com/plugins/superpowers
- 가이드: https://www.pasqualepillitteri.it/en/news/215/superpowers-claude-code-complete-guide
