# agent-deck 핸즈온 워크샵

[agent-deck](../../docs/terminal/agent-deck.md) ([asheshgoplani/agent-deck](https://github.com/asheshgoplani/agent-deck)) 은 **여러 AI 코딩 에이전트를 한 TUI 에서 굴리기 위한 세션 매니저**입니다. tmux 처럼 세션을 묶지만, 에이전트 상태(입력 대기/작업중/idle)를 자동 감지하고 MCP/스킬/비용/포크/conductor 같은 에이전트 워크플로우를 1급으로 다룹니다.

이 워크샵에서는 빈 폴더 두 개를 잡고:

- 한쪽엔 **Claude Code**, 다른 쪽엔 **Gemini CLI** 세션 추가
- 세션 **포크**로 같은 대화에서 갈래 두 개 동시에 시도
- **MCP** 한 번 정의해두고 세션별로 토글
- **Skill pool** 에 스킬 하나 넣고 attach/detach
- **비용 대시보드** 띄우기
- (선택) **conductor + Telegram** 으로 폰에서 받기

까지 짚어봅니다.

> 사전 학습:
> - [Claude Code 워크샵](../agents/claude-code-workshop.md)
> - (있으면) [Gemini CLI 워크샵](../agents/gemini-cli-workshop.md)
>
> 레퍼런스: [`docs/terminal/agent-deck.md`](../../docs/terminal/agent-deck.md)

---

## 0. 사전 준비

| 항목 | 확인 |
|---|---|
| Claude Code | `claude --version` |
| Gemini CLI (선택) | `gemini --version` |
| Git | `git --version` |
| macOS / Linux / WSL | agent-deck 동작 환경 |

(선택) Docker 샌드박스 / 리모트 / Telegram conductor 를 해보려면 각각 Docker / SSH / Telegram bot 토큰 필요.

---

## 1. 설치

### 1-1. 한 줄 설치 (가장 빠름)

```bash
curl -fsSL https://raw.githubusercontent.com/asheshgoplani/agent-deck/main/install.sh | bash
```

### 1-2. Homebrew

```bash
brew install asheshgoplani/tap/agent-deck
```

### 1-3. Go

```bash
go install github.com/asheshgoplani/agent-deck/cmd/agent-deck@latest
```

### 결과 확인

```bash
agent-deck --version
agent-deck doctor   # 환경 체크 (없으면 생략)
```

---

## 2. 첫 실행

```bash
agent-deck
```

처음 띄우면 빈 TUI 가 뜹니다. `?` 누르면 단축키 도움말. `q` 또는 `Ctrl+C` 로 종료.

설정 파일이 자동 생성됩니다:

```bash
ls ~/.agent-deck/
# config.toml   data/   skills/   logs/
cat ~/.agent-deck/config.toml
```

---

## 3. 첫 세션 — Claude Code

데모용 폴더 두 개 만들기:

```bash
mkdir ~/Desktop/ad-demo-a ~/Desktop/ad-demo-b
cd ~/Desktop/ad-demo-a && git init && echo "# A" > README.md && git add . && git commit -m "init"
cd ~/Desktop/ad-demo-b && git init && echo "# B" > README.md && git add . && git commit -m "init"
```

agent-deck TUI 열어서 `n` 누르면 **New Session** 다이얼로그. 또는 CLI 로:

```bash
agent-deck add ~/Desktop/ad-demo-a -c claude --name demo-a
agent-deck add ~/Desktop/ad-demo-b -c gemini --name demo-b   # gemini 없으면 -c claude 로 둘 다
```

### 결과 확인

TUI 에 두 줄이 뜨고 상태 인디케이터가 보입니다:

```
● demo-a   claude   ~/Desktop/ad-demo-a   running
○ demo-b   gemini   ~/Desktop/ad-demo-b   idle
```

`Enter` 로 attach → 해당 세션 (실은 tmux 안의 에이전트) 으로 진입. detach 는 tmux 기본 키 `Ctrl+B d` (또는 `[tmux] socket_name` 분리해 둔 경우 그 prefix).

`j/k` 로 위아래, `/` 로 fuzzy 검색.

---

## 4. 상태 자동 감지

`demo-a` 에 attach 해서 Claude 한테 길어 보이는 일을 시킵니다:

```
> 이 디렉토리에 hello world TS 프로젝트 셋업하고 npm 설치까지 해줘
```

작업이 진행되는 동안 `Ctrl+B d` 로 detach 하고 TUI 로 돌아오면 `demo-a` 의 인디케이터가 다음처럼 변합니다:

| 인디케이터 | 의미 |
|---|---|
| `● running` | 에이전트가 작업 중 |
| `◐ waiting` | 사용자 입력 기다리는 중 (승인 같은 것) |
| `○ idle` | 프롬프트 대기 |
| `✕ error` | 에러 / 중단 |

`@` 누르면 waiting 만, `!` 누르면 running 만 필터.

여러 세션을 띄워놓고도 어떤 게 지금 내 손이 필요한지 한눈에 보입니다.

---

## 5. 포크 — 같은 대화에서 갈래 두 개

`demo-a` 에 attach 해서 충분히 진행한 다음, detach 후 TUI 에서 `demo-a` 위에 `f`:

```
[Quick Fork] demo-a → demo-a-fork-1 (생성 중...)
```

또는 `F` 누르면 이름/그룹/디렉토리(워크트리 옵션) 다이얼로그.

### 결과 확인

```
● demo-a          claude   running
● demo-a-fork-1   claude   running   (forked from demo-a)
```

`Enter` 로 들어가면 **이전 대화 히스토리 그대로** 가져온 새 세션. 한쪽엔 "이 방법으로 가자", 다른 한쪽엔 "다른 접근으로 가자" 하면서 비교 가능.

> 💡 포크는 "큰 결정 직전 두 가지 시도" 에 좋습니다. 예: 라이브러리 A vs B, REST vs tRPC, 단일 파일 vs 분리.

---

## 6. MCP 토글

먼저 `~/.agent-deck/config.toml` 에 MCP 한 개 정의:

```toml
[mcp.exa]
command = "npx"
args = ["-y", "exa-mcp-server"]
env = { EXA_API_KEY = "${EXA_API_KEY}" }

[mcp.fs-docs]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/Documents"]
```

(테스트만 할 거면 둘 중 `fs-docs` 만 두는 게 키 없이 가능.)

TUI 에서 `demo-a` 위에 `m` → **MCP Manager**:

```
[MCP Manager — demo-a]
[ ] exa         GLOBAL
[x] fs-docs     LOCAL          ← Space 로 토글
```

`Tab` 으로 LOCAL/GLOBAL 전환. LOCAL 은 이 세션만, GLOBAL 은 모든 세션 기본.

토글하면 자동으로 세션 재시작(`r`) 또는 묻기.

### 결과 확인

attach 해서:

```
> /mcp
```

또는 (Gemini 면 그쪽 명령) — 방금 추가/제거한 MCP 가 등록된 게 보여야 합니다.

CLI 로도 가능:

```bash
agent-deck mcp attach demo-a fs-docs --scope local --restart
agent-deck mcp detach demo-a fs-docs
```

> 💡 동시에 5개 세션이 같은 MCP 를 쓴다면 `[mcp] pool_all = true` 켜두세요. 프로세스 한 개 + Unix 소켓으로 공유 → 메모리 85~90% 절감.

---

## 7. Skill Pool — 세션별 attach/detach

[Superpowers](./superpowers-workshop.md) 같은 스킬을 통째로 깔지 않고 골라 쓰고 싶을 때 유용.

풀 디렉토리 만들고 스킬 하나 넣기:

```bash
mkdir -p ~/.agent-deck/skills/pool/team-pr-checklist
cat > ~/.agent-deck/skills/pool/team-pr-checklist/SKILL.md <<'EOF'
---
name: team-pr-checklist
description: Use before creating any PR — enforces our team's quality gates
---

PR 만들기 전 다음 모두 확인:
1. `pnpm test` 통과
2. CHANGELOG 업데이트
3. 마이그레이션 있으면 staging dry-run
4. 한국어 PR 제목 + 영어 본문
EOF
```

TUI 에서 `demo-a` 위에 `s` → **Skills Manager**:

```
[Skills Manager — demo-a]
[ ] team-pr-checklist     pool
```

Space 로 attach. attach 하면:

1. `~/Desktop/ad-demo-a/.agent-deck/skills.toml` 에 기록
2. `~/Desktop/ad-demo-a/.claude/skills/team-pr-checklist/SKILL.md` 로 materialize
3. (옵션) 세션 재시작

### 결과 확인

```bash
ls ~/Desktop/ad-demo-a/.claude/skills/
# team-pr-checklist
cat ~/Desktop/ad-demo-a/.agent-deck/skills.toml
```

attach 한 세션에서 PR 만들기 시도:

```
> 이 작업 PR 만들어줘
```

→ Claude 가 위 체크리스트를 따라가야 합니다.

CLI 도 동일:

```bash
agent-deck skill attach demo-a team-pr-checklist --source pool --restart
agent-deck skill detach demo-a team-pr-checklist
```

> 💡 이 패턴이면 팀 공통 스킬은 **레포에 커밋된 `.claude/skills/`**, 개인 도구상자는 **풀**, 둘이 공존합니다.

---

## 8. 그룹 / 프로파일

세션이 늘어나면 그룹으로 묶기. TUI 에서 세션 위 `M` → 그룹 선택 / 새 그룹.

```
work/
  demo-a
  demo-b
oss/
  side-project
```

프로파일은 Claude config 디렉토리를 분리. 회사용 / 개인용 인증 키를 섞지 않을 때 유용.

`~/.agent-deck/config.toml`:

```toml
[claude]
config_dir = "~/.claude"

[profiles.work.claude]
config_dir = "~/.claude-work"
env_file   = "~/.config/agent-deck/work.env"
```

```bash
agent-deck -p work add ~/Desktop/work-proj -c claude
```

### 결과 확인

```
[work profile]
● work-proj   claude   ~/Desktop/work-proj
```

키 / 메모리 / 슬래시 커맨드가 `~/.claude-work` 와 묶입니다.

---

## 9. 비용 대시보드

Claude Code 가 한참 일한 다음 TUI 에서 `$`:

```
[Cost Dashboard]
Today    : $1.21      Limit : $30/day  (warn 80%)
Week     : $4.30
Month    : $12.05

By session today
  demo-a       $0.62   Sonnet  41k in / 22k out
  demo-a-fork  $0.41   Sonnet  28k in / 14k out
  demo-b       $0.18   Gemini Flash  ...
```

웹 대시보드:

```bash
agent-deck web
# http://127.0.0.1:8420
```

브라우저에서 Chart.js 그래프 / 모델별 / 세션별 드릴다운.

읽기전용 모드 (공유 / 데모용):

```bash
agent-deck web --read-only --listen 0.0.0.0:9000
```

### 한도 설정

```toml
[costs]
daily_limit_usd  = 30
monthly_limit_usd = 500
warn_at_pct = 80
```

80% 도달하면 경고 알림, 100% 도달하면 새 작업 거부.

---

## 10. (선택) Worktree 로 격리 작업

큰 변경을 시작할 때 메인 폴더와 분리:

```bash
agent-deck add ~/Desktop/ad-demo-a -c claude \
  --worktree feature/themes --new-branch \
  --name demo-a-themes
```

내부적으로:

```bash
git worktree add ../ad-demo-a-feature-themes -b feature/themes
```

`~/Desktop/ad-demo-a/.agent-deck/worktree-setup.sh` 가 있으면 워크트리 생성 직후 자동 실행. `.env` 복사 / 의존성 설치 같은 데 유용.

작업 끝나면:

```bash
agent-deck worktree finish demo-a-themes   # merge → cleanup → 세션 삭제
```

---

## 11. (선택) Docker 샌드박스 / `try`

신뢰 못 할 PR 검증, 잘못 건드리면 시스템 망가질 수 있는 일은 컨테이너에서:

```toml
[docker]
default_enabled = false   # 켜면 모든 신규 세션이 sandboxed
mount_ssh = true
auto_cleanup = true
```

CLI 로 1회성:

```bash
agent-deck try "이 의존성 업데이트 PR 만들어봐"
```

→ 격리 컨테이너 안에서 Claude 가 작업, 끝나면 자동 정리.

샌드박스 세션 위에서 `T` 누르면 컨테이너 셸 진입.

---

## 12. (선택) Conductor + Telegram

영속 에이전트가 다른 세션 상태를 감시하다가 자동 응답 / 사용자에게 escalate.

```bash
agent-deck conductor setup ops --description "Ops monitor for waiting/error sessions"
```

프롬프트가 시작되면 Telegram bot token 또는 Slack pairing 입력. 텔레그램에선 [@BotFather](https://t.me/BotFather) 로 봇 만들고 토큰만 붙여넣기.

연결 끝나면:

- `demo-a` 가 waiting 상태에 빠지면 conductor 가 감지
- 자신 있는 응답이면 자동 (예: "yes" / "continue")
- 모호하면 텔레그램으로 푸시 → 폰에서 답장

watcher 추가로 외부 이벤트도 받을 수 있음:

```bash
agent-deck watcher create github --name gh-alerts --secret $WEBHOOK_SECRET
agent-deck watcher start gh-alerts
```

자세히는 repo 의 `CONDUCTOR.md`, `WATCHERS.md`.

---

## 13. (선택) Remote 인스턴스

리모트 dev 박스에 agent-deck 자동 설치 + 로컬 TUI 에 같이 띄우기:

```bash
agent-deck remote add dev user@dev-box --auto-install
agent-deck remote sessions dev
agent-deck remote attach dev big-build
```

```
[remote: dev]
  ● big-build  claude  /home/user/big-build  running
```

로컬 단축키 그대로 사용. SSH 끊겨도 리모트 데몬은 살아 있음.

---

## 14. 트러블슈팅

### `agent-deck: command not found`

PATH 문제. Homebrew 면 `brew link agent-deck`, 수동 설치면 `~/.local/bin` 또는 설치 로그 마지막 줄 확인.

### tmux 세션이 내 개인 tmux 와 섞임

```toml
[tmux]
socket_name = "agent-deck"
```

별도 tmux 서버에서 격리됨.

### MCP attach 했는데 Claude 가 못 봄

- 세션 재시작 됐는지 확인 (`r`)
- LOCAL/GLOBAL 스코프 헷갈리지 않게 — 일단 LOCAL 로
- `~/.agent-deck/logs/mcp-*.log` 에서 stderr 확인

### 비용이 안 잡힘

Claude Code 훅으로 자동 수집되므로 `~/.claude/settings.json` 에 agent-deck 가 박은 훅이 그대로 있는지:

```bash
cat ~/.claude/settings.json | grep agent-deck
```

지워졌으면 `agent-deck doctor` 또는 재설치.

### 포크가 너무 많이 쌓임

```bash
agent-deck session remove <name>   # 종료된 세션 정리
agent-deck session prune --idle-for 7d
```

---

## 15. 전체 흐름 요약

```
[설치] curl ... | bash    또는    brew install asheshgoplani/tap/agent-deck
   ↓
[TUI] agent-deck
   ↓
[세션 추가] add . -c claude   add . -c gemini   ...
   ↓
[작업] Enter 로 attach → 일반 에이전트처럼 사용
   ↓
[멀티 세션 굴리기]
   ◐ waiting / ● running / ○ idle 자동 감지
   f / F : 포크 (대화 분기)
   m     : MCP per-session 토글
   s     : Skill pool attach/detach
   $     : 비용 대시보드
   M     : 그룹화
   /  G  : 검색 (세션 / 모든 대화)
   T     : 컨테이너 셸
   ↓
[고급]
   conductor → 폰/텔레그램으로 escalate
   watcher   → GitHub / Slack / ntfy 이벤트 라우팅
   remote    → SSH 너머 인스턴스 통합
   worktree  → 브랜치별 격리
```

---

## 16. 다른 도구와 함께 쓰기

| 조합 | 효과 |
|---|---|
| agent-deck + [Superpowers](./superpowers-workshop.md) | Superpowers 스킬을 풀에 두고 프로젝트별 attach. 가벼운 프로젝트엔 안 깔기 |
| agent-deck + [GSD](./gsd-workshop.md) | GSD 의 phase 기반 컨텍스트와 agent-deck 의 워크트리 격리 결합 |
| agent-deck + [gstack](./gstack-workshop.md) | gstack 역할 스킬을 풀에 두고 SaaS 풀사이클에서 attach/detach |
| agent-deck + [tmux](../../docs/terminal/tmux.md) | agent-deck 가 내부에서 tmux 사용 — 셋이 충돌 안 함. `socket_name` 분리만 |
| agent-deck + [cmux](../../docs/terminal/cmux.md) | 로컬 GUI 는 cmux, 헤드리스 / 리모트 / 다중 프로젝트는 agent-deck |

---

## 17. 더 보기

- 레퍼런스: [`docs/terminal/agent-deck.md`](../../docs/terminal/agent-deck.md)
- 메인 repo: https://github.com/asheshgoplani/agent-deck
- 공식 사이트: https://asheshgoplani.github.io/agent-deck/
- README: https://github.com/asheshgoplani/agent-deck/blob/main/README.md
- Releases: https://github.com/asheshgoplani/agent-deck/releases
- 관련 워크샵:
  - [Claude Code 워크샵](../agents/claude-code-workshop.md)
  - [Gemini CLI 워크샵](../agents/gemini-cli-workshop.md)
  - [Superpowers 워크샵](./superpowers-workshop.md)
  - [GSD 워크샵](./gsd-workshop.md)
