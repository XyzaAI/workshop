# Superpowers

[obra/superpowers](https://github.com/obra/superpowers) (v5.0.7+) — Claude Code에 **Skill 기반의 동작 디시플린**을 입히는 확장 모음. obra(Jesse Vincent) 가 만들어 공개한 것이 베이스이며 2026-01-15 부터 **공식 Anthropic 마켓플레이스** 에 등재.

> 핸즈온 가이드: [`../../workshops/extensions/superpowers-workshop.md`](../../workshops/extensions/superpowers-workshop.md). 이 문서는 **레퍼런스**.

---

## 무엇을 해주나

기본 Claude Code는 "프롬프트가 시키는 대로" 동작합니다. Superpowers는 **메타 워크플로우**를 강제합니다:

- "버그 발견하면 무조건 systematic-debugging 절차 따라가"
- "구현 시작 전엔 brainstorming부터"
- "테스트 없이 구현 코드 쓰지 마 (TDD)"
- "큰 작업은 plan 먼저 작성해서 사용자 승인받기"

이런 디시플린이 **스킬 단위**로 모듈화되어 있고, 적절한 상황에 자동/강제 호출됩니다.

---

## 대표 스킬들

| 스킬 | 언제 발동 | 효과 |
|---|---|---|
| `using-superpowers` | 모든 대화 시작 | 다른 스킬을 인식·사용하게 함 |
| `brainstorming` | 새 기능/리팩터링 시작 | 구현 전 요구사항/설계 합의 |
| `writing-plans` | 멀티스텝 작업 | 구조화된 plan 문서 작성 |
| `executing-plans` | plan을 실제 실행 | 체크포인트 기반 실행 |
| `test-driven-development` | 구현 시작 | 빨강→초록→리팩터 강제 |
| `systematic-debugging` | 버그/실패 발견 | 가설 수립 → 검증 루프 |
| `verification-before-completion` | "완료" 주장 직전 | 검증 명령 실행 의무화 |
| `requesting-code-review` | 작업 완료 후 | 자동 리뷰 트리거 |
| `using-git-worktrees` | 격리 필요한 작업 | worktree로 분리 작업 |
| `dispatching-parallel-agents` | 독립 작업 2개 이상 | 서브에이전트 병렬 실행 |

---

## 설치

### 공식 마켓플레이스 (추천)

Claude Code 안에서:

```
> /plugin install superpowers@claude-plugins-official
```

또는 [공식 페이지](https://claude.com/plugins/superpowers) 에서 설치.

### 커뮤니티 마켓플레이스 (최신 dev)

```
> /plugin marketplace add obra/superpowers-marketplace
> /plugin install superpowers@superpowers-marketplace
```

### 직접 clone (로컬 개발용)

```bash
git clone https://github.com/obra/superpowers ~/.claude/plugins/superpowers
```

스킬은 `~/.claude/plugins/superpowers/skills/<skill-name>/SKILL.md` 구조로 깔립니다. 자체 스킬은 `~/.claude/skills/<name>/SKILL.md`.

설치 후 Claude Code 재시작 → `/plugin` 으로 `superpowers v5.0.7+` 확인.

---

## SKILL.md 구조

각 스킬은 frontmatter + 본문으로 이뤄집니다:

```markdown
---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

## 절차

1. 재현 가능한 최소 케이스 만들기
2. 가설 적기 (3개 이상)
3. 가설별 검증 방법 정하기
4. 검증 실행 → 가설 좁히기
5. 근본 원인 확정 후 수정
...
```

`description`이 매우 중요 — Claude가 이 텍스트만 보고 "지금 호출할지" 판단합니다.

---

## 직접 만들기

자기 팀/프로젝트만의 디시플린을 스킬로 만들면 다음 세션부터 자동 적용됩니다.

```bash
mkdir -p ~/.claude/skills/our-pr-checklist
```

`SKILL.md`:

```markdown
---
name: our-pr-checklist
description: Use before creating any PR — enforces our team's quality gates
---

PR 만들기 전 다음을 모두 확인:

1. `pnpm test` 통과
2. `pnpm lint` 통과
3. 변경된 API에 OpenAPI 스펙 업데이트됨
4. 마이그레이션 있으면 staging에서 dry-run 완료
5. PR 제목 = conventional commit 형식
```

자세한 작성법은 `superpowers:writing-skills` 스킬 참고.

---

## 효과 / 한계

**효과**:
- 같은 실수 반복 방지 (특히 TDD/debugging)
- 큰 작업의 구조 강제 → 결과물 일관성 ↑
- 팀 내 디시플린 코드화 — 신규 멤버 온보딩에 유용

**한계**:
- 잘 안 짜면 오버헤드 (사소한 작업에 매번 brainstorming 강제하는 식)
- description이 두루뭉술하면 잘못된 상황에 호출됨
- Claude Code 외에선 안 돌아감 (다른 에이전트는 호환 X)

---

## 참고

- 메인: https://github.com/obra/superpowers
- Lab (실험적 스킬): https://github.com/obra/superpowers-lab
- Marketplace: https://github.com/obra/superpowers-marketplace
- 공식 페이지: https://claude.com/plugins/superpowers
- DeepWiki: https://deepwiki.com/obra/superpowers
- 가이드: https://www.pasqualepillitteri.it/en/news/215/superpowers-claude-code-complete-guide
- 핸즈온 워크샵: [`../../workshops/extensions/superpowers-workshop.md`](../../workshops/extensions/superpowers-workshop.md)
- 관련 문서: [`../tips/productivity.md`](../tips/productivity.md)
