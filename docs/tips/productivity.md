# 생산성 팁

도구별 가이드는 각 문서에 있고, 여기는 **공통적으로 효과 큰 패턴**들 모음.

대부분 Claude Code 기준이지만 다른 에이전트에도 응용 가능.

---

## 1. 권한을 풀어줘라 (가장 효과 큼)

기본 모드에선 매번 묻습니다 — 클릭/타이핑 비용이 어마어마. **자주 쓰는 안전한 명령은 미리 허용**.

```json
// ~/.claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(pnpm test *)",
      "Bash(pnpm lint *)",
      "Bash(pnpm build)",
      "Bash(rg *)",
      "Bash(ls *)",
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(git push --force *)",
      "Bash(git reset --hard *)"
    ]
  }
}
```

지금까지 자기가 어떤 명령을 자주 승인했는지 트랜스크립트에서 뽑는 스킬 (`fewer-permission-prompts`) 도 있음.

---

## 2. CLAUDE.md / GEMINI.md 적극 활용

세션마다 하던 설명이 보이면 메모리에 박을 신호.

```markdown
# 프로젝트 규칙
- 패키지 매니저: pnpm
- 테스트: vitest, *.test.ts, 통과 안 되면 절대 commit 금지
- 마이그레이션: shadow DB에서 dry-run 먼저
- 커밋 메시지: conventional commits, 한국어 가능
- 외부 API 호출은 src/clients/ 통해서만
- 로깅은 winston, console.log 금지
```

좋은 CLAUDE.md 의 조건:
- 짧고 단호 (모호한 권유는 무시됨)
- 자주 어기는 것만 (다 적으면 다 무시됨)
- "왜"를 가끔 추가 (조건부로 어길 줄 알게)

---

## 3. 모델을 작업 성격으로 골라라

| 작업 | 추천 모델 |
|---|---|
| 큰 변경 / 추론 무거움 / 보안 리뷰 | Opus 4.7, o3, Gemini 2.5 Pro |
| 일반 구현 / 리팩터 / 디버깅 | Sonnet 4.6, GPT-5 |
| TDD 루프 / 보일러플레이트 / 단순 편집 | Haiku 4.5, grok-code-fast |
| 큰 코드베이스 통째 읽기 | Gemini (1M~2M 컨텍스트) |
| 비용 0 / 오프라인 | Ollama + Hermes-3 등 로컬 |

`/model` 로 도중에 바꿀 수 있음. 기획은 Opus, 구현은 Sonnet 같은 식 흔함.

---

## 4. 세션 짧게 끊고 자주 새로 시작

긴 대화 = 토큰 낭비 + 모델 혼동. 작업 단위로 끊고:

- `/clear` — 같은 세션에서 컨텍스트만 비우기
- 새 터미널 / `claude` 새로 — 완전 초기화
- `/compact` — 끊긴 가는데 컨텍스트는 살리고 싶을 때

루틴: **한 작업 = 한 세션**. 끝내면 commit 후 새 세션.

---

## 5. worktree로 격리

큰 변경이나 실험은 메인 작업 디렉토리에서 분리:

```bash
git worktree add ../myproject-feat-x -b feat/x
cd ../myproject-feat-x
claude
```

Claude Code의 `superpowers:using-git-worktrees` 스킬을 쓰면 자동화. 메인은 깨끗하게 유지하면서 병렬로 실험 가능.

cmux / amux 같은 도구는 이걸 더 쉽게 해줌.

---

## 6. 하나의 프롬프트, 여러 모델

같은 작업을 여러 모델에 동시에 시키고 결과 비교 — 가끔 답이 갈리는 게 가장 가치 있는 신호.

도구별 방법:
- **Crush** — 같은 세션 모델 갈아끼우기
- **OpenCode (server)** — 클라이언트 둘 띄워 같은 세션 attach
- **cmux / 멀티 에이전트** — pane 두 개에 다른 에이전트 띄우고 동시 입력 (`tmux setw synchronize-panes on`)

---

## 7. 슬래시 커맨드를 적극 만들어라

3번 이상 같은 패턴 프롬프트가 나오면 슬래시 커맨드로:

```markdown
<!-- ~/.claude/commands/review.md -->
---
description: PR 리뷰
---

현재 브랜치의 변경사항을 다음 관점으로 리뷰해줘:
1. 보안 (XSS/SQLi/secret 노출)
2. 테스트 커버리지
3. 네이밍/타입 일관성
4. 성능 회귀 가능성
끝나면 한국어로 GitHub 코멘트 형태로 출력.
```

이제 어디서든 `/review` 한 번에 호출.

---

## 8. 서브에이전트로 메인 컨텍스트 보호

큰 리서치 / 광범위 탐색은 서브에이전트한테 시켜서 메인 컨텍스트는 깨끗하게 유지.

```markdown
<!-- ~/.claude/agents/explorer.md -->
---
name: explorer
description: 큰 코드베이스 탐색 / 정리된 요약만 메인에 전달
tools: Read, Grep, Glob
---

당신은 탐색 전문가. 사용자가 묻는 질문에 대한 답을 코드베이스에서 찾아서
**핵심만 200단어 이내**로 정리해서 반환하세요.
```

긴 grep 결과가 메인 컨텍스트를 잡아먹는 걸 막음.

---

## 9. 알림 셋업 (멀티태스킹의 시작)

작업 끝났는지 모니터 보고 있어야 한다 = 멀티태스킹 실패.

자세히는 [`./notifications.md`](./notifications.md). 최소한:

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Glass.aiff" }]
    }],
    "Notification": [{
      "matcher": "",
      "hooks": [{ "type": "command", "command": "say -v Yuna '입력 필요'" }]
    }]
  }
}
```

---

## 10. 자주 어기는 안티패턴

### ❌ 한 세션에서 모든 일 다 하기
컨텍스트 누적되어 점점 멍청해짐. 끊자.

### ❌ 매 명령 매번 직접 승인
`permissions.allow` 등록. 시간/돈/멘털 다 절약.

### ❌ "고쳐줘" 만 던지고 결과 검증 안 함
에이전트는 자신감 있게 잘못된 답을 내놓을 수 있음. 테스트로 검증, 아니면 `superpowers:verification-before-completion` 같은 스킬로 강제.

### ❌ 너무 많은 hooks / 스킬 / 메모리 박기
Hooks 잔뜩 박아두면 세션 시작/도구 호출마다 느려짐. 메모리 100줄 넘으면 모델이 무시. **최소주의**.

### ❌ 프로덕션 키 평문 노출
`.env` / 환경변수만 사용. settings.json / CLAUDE.md에 시크릿 박지 마.

### ❌ 같은 실수 반복 — 메모리 안 씀
모델이 같은 걸 또 틀리면 그게 메모리에 박을 신호. 방치하면 다음에 또 그런다.

---

## 11. 권장 디렉토리 / 파일 구조

```
~/.claude/
├── CLAUDE.md              # 글로벌 메모리 (당신이 누구, 어떻게 일하는지)
├── settings.json          # 권한, 환경, 훅 (전체 적용)
├── commands/              # 자주 쓰는 슬래시 커맨드
│   ├── review.md
│   ├── ship.md
│   └── debug.md
├── agents/                # 서브에이전트
│   └── explorer.md
└── skills/                # 스킬 (선택)

<project>/
├── CLAUDE.md              # 프로젝트 규칙 (커밋 — 팀과 공유)
├── .claude/
│   ├── settings.json      # 프로젝트 설정 (커밋)
│   ├── settings.local.json # 개인 설정 (.gitignore)
│   ├── commands/          # 프로젝트 전용 커맨드
│   └── agents/
└── ...
```

---

## 12. 한 번 배워두면 평생 쓰는 단축

| 도구 | 단축 |
|---|---|
| Claude Code | `Shift+Tab` 권한 모드, `Esc` 중단, `!cmd` 셸 실행, `↑/↓` 히스토리 |
| 터미널 | tmux/cmux로 pane 분할 / 세션 detach |
| 에디터 | 파일 경로 → 에디터로 점프 (`code path:line`) |
| Git | `gh pr view`, `gh issue list`, `git log --oneline -20` |

---

## 더 보기

- 알림: [`./notifications.md`](./notifications.md)
- Hooks: [`./hooks.md`](./hooks.md)
- MCP: [`./mcp-setup.md`](./mcp-setup.md)
- Claude Code 레퍼런스: [`../agents/claude-code.md`](../agents/claude-code.md)
- Superpowers (메서드론 디시플린): [`../extensions/superpowers.md`](../extensions/superpowers.md)
