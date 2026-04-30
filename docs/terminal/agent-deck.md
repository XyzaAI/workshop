# agent-deck

[asheshgoplani/agent-deck](https://github.com/asheshgoplani/agent-deck) — **AI 코딩 에이전트 전용 터미널 세션 매니저**. Claude Code / Codex / Gemini CLI / OpenCode / Cursor 를 한 TUI 안에서 함께 띄우고, 상태/포크/MCP/스킬/비용까지 한 자리에서 관리합니다.

> tmux 류는 "터미널 일반 멀티플렉서", agent-deck 은 "**에이전트 전용**". 세션 상태 표시·MCP 토글·스킬 attach·conductor 같이 에이전트 굴릴 때만 필요한 기능들이 1급 시민입니다.
>
> 핸즈온 가이드: [`../../workshops/extensions/agent-deck-workshop.md`](../../workshops/extensions/agent-deck-workshop.md). 이 문서는 **레퍼런스**.

---

## 왜 만들어졌나

여러 프로젝트에서 동시에 에이전트를 굴릴 때 일반 터미널은:

- 어떤 세션이 입력 기다리는지 즉시 안 보임
- 같은 대화에서 갈래 두 개 시도(fork) 하기 번거로움
- 프로젝트마다 MCP 끄고 켜기 위해 `.mcp.json` 손으로 편집
- 토큰/비용 추적이 에이전트마다 따로 놀음
- 세션이 10개 넘어가면 어디 뭐 띄웠는지 잊어버림

agent-deck 은 위를 한 TUI 에 합치고, 아래에 별도 데몬(`agent-deckd`) 이 돌면서 상태를 추적·라우팅합니다.

---

## 핵심 개념

| 개념 | 설명 |
|---|---|
| **Session** | 하나의 디렉토리 + 에이전트 조합. 내부적으론 tmux 세션 하나로 보존 |
| **Group** | 세션을 논리적으로 묶기 (예: `work/`, `oss/` 트리) |
| **Profile** | Claude config 디렉토리 / API key 분리 (`~/.claude-work` vs `~/.claude-personal`) |
| **MCP Pool** | MCP 프로세스를 Unix 소켓으로 공유 — 메모리 85~90% 절감 |
| **Skill Pool** | `~/.agent-deck/skills/pool` 에 모은 스킬을 세션별로 attach/detach |
| **Conductor** | 모든 세션을 감시하는 영속 에이전트. 자신 있게 자동 응답 / 모호하면 사용자 호출 (Telegram/Slack 연동) |
| **Watcher** | 외부 이벤트(GitHub webhook / ntfy / Slack / 웹훅) 를 conductor 에게 라우팅 |
| **Worktree** | git worktree 자동 생성·정리. 세션 끝낼 때 머지·삭제까지 일괄 |
| **Sandbox** | 세션을 Docker 컨테이너에 격리. `T` 누르면 컨테이너 셸 진입 |
| **Remote** | SSH 너머의 다른 머신 agent-deck 인스턴스 세션을 로컬 TUI 에 같이 띄움 |

---

## 설치

### 한 줄 설치 (공식)

```bash
curl -fsSL https://raw.githubusercontent.com/asheshgoplani/agent-deck/main/install.sh | bash
```

macOS / Linux / WSL.

### Homebrew

```bash
brew install asheshgoplani/tap/agent-deck
```

### Go

```bash
go install github.com/asheshgoplani/agent-deck/cmd/agent-deck@latest
```

### 소스

```bash
git clone https://github.com/asheshgoplani/agent-deck.git
cd agent-deck
make install
```

설치 후:

```bash
agent-deck --version
agent-deck            # TUI 실행
```

---

## 기본 명령어

```bash
agent-deck                                  # TUI 실행
agent-deck add . -c claude                  # 현재 폴더에 Claude 세션 추가
agent-deck add . -c gemini                  # Gemini CLI 로
agent-deck add . -c codex                   # Codex 로
agent-deck add . -c opencode                # OpenCode 로
agent-deck session fork my-proj             # 세션 포크
agent-deck session remove my-proj           # 종료된 세션 정리
agent-deck mcp attach my-proj exa           # MCP 붙이기
agent-deck skill attach my-proj docs --source pool --restart
agent-deck web                              # 웹 UI (기본 포트 8420)
agent-deck try "task description"           # Docker 샌드박스에서 1회성 실행
agent-deck remote add dev user@dev-box      # 리모트 인스턴스 등록
agent-deck remote attach dev my-session
agent-deck conductor setup ops --description "Ops monitor"
agent-deck watcher create github --name gh-alerts --secret $WEBHOOK_SECRET
agent-deck update                           # standalone 업데이트
agent-deck feedback                         # 피드백 보내기
```

---

## TUI 단축키

| 키 | 동작 |
|---|---|
| `Enter` | 세션 attach (해당 tmux 세션으로 진입) |
| `n` | 새 세션 |
| `f` / `F` | 포크 (빠르게 / 다이얼로그) |
| `m` | MCP Manager |
| `s` | Skills Manager |
| `$` | Cost Dashboard |
| `M` | 세션을 그룹으로 이동 |
| `S` | 설정 |
| `/` | 세션 안 fuzzy 검색 |
| `G` | 모든 Claude 대화에 대한 글로벌 검색 |
| `r` | 세션 재시작 |
| `d` | 삭제 |
| `T` | 컨테이너 셸 (sandboxed 세션만) |
| `Ctrl+E` | 피드백 다이얼로그 |
| `?` | 전체 도움말 |
| `j` / `k` | 위/아래 (vim 스타일) |
| `gg` / `G` | 맨 위 / 맨 아래 |
| `1`–`9` | N번째 root group 으로 점프 |
| `Alt+j` / `Alt+k` | 그룹 안에서만 다음/이전 |
| `Alt+1`–`9` | 그룹 안 N번째 세션 |

상태 필터: `!` running / `@` waiting / `#` idle / `$` error.

---

## 설정 파일

### 메인: `~/.agent-deck/config.toml`

자주 쓰는 섹션:

```toml
# ── Claude 전역 설정 ─────────────────────────
[claude]
config_dir = "~/.claude"
# env_file = "~/.config/agent-deck/claude.env"
# allow_dangerous_mode = false

# ── 프로파일별 Claude 분리 ───────────────────
[profiles.work.claude]
config_dir = "~/.claude-work"
env_file   = "~/.config/agent-deck/work.env"

# ── MCP 정의 (한 곳에 모아 두고 세션별 토글) ─
[mcp.exa]
command = "npx"
args = ["-y", "exa-mcp-server"]
env = { EXA_API_KEY = "${EXA_API_KEY}" }

[mcp.notion]
type = "http"
url = "https://mcp.notion.com"

# ── MCP 프로세스 풀링 ────────────────────────
[mcp]
pool_all = true                     # 모든 MCP 풀 공유 (메모리 ↓)

# ── 비용 추적 ────────────────────────────────
[costs]
daily_limit_usd  = 30
monthly_limit_usd = 500
warn_at_pct = 80

# ── tmux 격리 ────────────────────────────────
[tmux]
socket_name = "agent-deck"          # 개인 tmux와 분리

# ── Docker 샌드박스 ──────────────────────────
[docker]
default_enabled = false
mount_ssh = true
auto_cleanup = true

# ── 그룹별 오버라이드 ────────────────────────
[groups."work"]
[groups."work".claude]
config_dir = "~/.claude-work"

# ── 리모트 인스턴스 ──────────────────────────
[remotes.dev]
ssh_host = "user@dev-box"
auto_install = true

# ── 알림 ─────────────────────────────────────
[notifications]
on_waiting = true
on_error   = true
```

### 세션별: `<프로젝트>/.agent-deck/skills.toml`

```toml
[skills]
attached = ["systematic-debugging", "writing-plans"]
```

### 워크트리 후처리: `<프로젝트>/.agent-deck/worktree-setup.sh`

워크트리 만들 때마다 실행되는 스크립트. `.env` 복사, 의존성 설치, `pnpm install` 같은 것.

```bash
#!/usr/bin/env bash
set -e
cp ../<원본>/.env ./.env
pnpm install
```

---

## 호환 에이전트

| 에이전트 | 통합 수준 |
|---|---|
| Claude Code | Full — 상태 / MCP 토글 / 포크 / resume / 스킬 |
| Gemini CLI | Full — 상태 / MCP / resume |
| OpenCode | 상태 감지 + 그룹 / 검색 |
| Codex | 상태 감지 + conductor 가능 |
| Cursor (terminal) | 상태 감지 + 그룹 |
| 커스텀 | `[tools.<name>]` 섹션으로 추가 |

커스텀 도구 등록 예:

```toml
[tools.aider]
command = "aider"
args = ["--stream"]
patterns = { running = "Tokens:", waiting = "^>" }
```

---

## MCP 관리 (이게 진짜 강점 중 하나)

기본 흐름:

1. `~/.agent-deck/config.toml` 의 `[mcp.<name>]` 에 MCP 정의 한 번
2. TUI 에서 세션 위에 `m` → MCP Manager
3. `Space` 로 토글, `Tab` 으로 LOCAL(이 세션만) ↔ GLOBAL 전환
4. attach/detach 하면 자동으로 세션 재시작

**Pool**: `[mcp] pool_all = true` 로 두면 한 MCP 프로세스를 모든 세션이 Unix 소켓으로 공유 — 동시 세션 10개여도 MCP 프로세스는 1개. 메모리 85~90% 절감, 시작 즉시 응답.

CLI 로:

```bash
agent-deck mcp attach my-proj exa --scope local --restart
agent-deck mcp detach my-proj exa
```

---

## Skills 관리

두 단계 시스템:

- **User-level**: `<프로젝트>/.claude/skills/<name>/SKILL.md` — git tracked, 팀 공유
- **Pool**: `~/.agent-deck/skills/pool/<name>/SKILL.md` — 개인 보관, 세션별 attach

`s` 누르면 Skills Manager. attach 하면:
1. `.agent-deck/skills.toml` 에 기록
2. 풀에서 `.claude/skills/` 로 materialize
3. (옵션) 세션 재시작

CLI:

```bash
agent-deck skill attach my-proj systematic-debugging --source pool --restart
agent-deck skill detach my-proj systematic-debugging
```

[Superpowers](../extensions/superpowers.md) 풀의 일부만 골라 쓰는 식으로 자주 활용.

---

## Conductor / Watcher

**Conductor** = 항상 떠 있는 영속 에이전트. 다른 세션이 입력 기다리거나 에러 나면 감지하고 자동 응답할지 사용자에게 escalate 할지 판단.

```bash
agent-deck conductor setup ops --description "Ops monitor"
# 프롬프트 따라 Telegram bot token / Slack pairing 설정
```

연결 후 폰에서 텔레그램 메시지로 conductor 와 대화 가능 → conductor 가 적절한 세션에 명령 dispatch.

**Watcher** = 외부 이벤트를 conductor 한테 떨어뜨리는 라우터.

| 어댑터 | 용도 |
|---|---|
| webhook | 일반 HTTP POST 수신 |
| github | repo webhook (HMAC-SHA256 검증) |
| ntfy | ntfy.sh 푸시 알림 → conductor |
| slack | Cloudflare Worker 브릿지 경유 |

```bash
agent-deck watcher create github --name gh-alerts --secret $WEBHOOK_SECRET
agent-deck watcher start gh-alerts
```

자세히는 repo 의 `WATCHERS.md` / `CONDUCTOR.md` 참고.

---

## 비용 대시보드

`$` 누르면 TUI 안에서:

```
Today    : $4.21    Week : $18.30   Month : $63.05
Limit    : $30/day  Warn at 80%

Top sessions today:
  ml-pipeline  $1.80  Opus   24k in / 18k out
  workshop-ko  $1.05  Sonnet 41k in / 22k out
  ...
```

지원 모델 13종 (Claude Opus / Sonnet / Haiku, Gemini Pro / Flash, GPT-4o variants 등). Claude Code 훅 통해 자동 수집.

웹 대시보드 (`agent-deck web`) 에선 Chart.js 그래프로 추세 / 드릴다운.

---

## Worktree / Sandbox

### Worktree (브랜치별 격리 작업)

```bash
agent-deck add . -c claude --worktree feature/a --new-branch
agent-deck add . -c claude --worktree feature/b --location subdirectory
# 끝나면
agent-deck worktree finish "My Session"   # merge → cleanup → 세션 삭제
```

`.agent-deck/worktree-setup.sh` 가 있으면 워크트리 생성 직후 실행.

### Docker Sandbox

```toml
[docker]
default_enabled = true
mount_ssh = true
```

세션 위에서 `T` 누르면 컨테이너 셸. 1회성 격리 실행:

```bash
agent-deck try "이 의존성 업데이트 PR 만들어봐"
```

---

## Remote 인스턴스

리모트 dev-box 에서도 agent-deck 를 띄워두고 로컬 TUI 에 같이 보이기:

```bash
agent-deck remote add dev user@dev-box --auto-install
agent-deck remote sessions dev
agent-deck remote attach dev my-session
```

리모트 세션은 로컬 세션 옆에 마크 표시되어 같은 단축키로 다룸.

---

## tmux / cmux 와 비교

| 항목 | agent-deck | [tmux](./tmux.md) | [cmux](./cmux.md) |
|---|---|---|---|
| 형태 | TUI (Go + Bubble Tea) | CLI | macOS GUI 앱 |
| 플랫폼 | macOS / Linux / WSL | 모든 \*nix | macOS only |
| 대상 | AI 에이전트 전용 | 모든 셸 | AI 에이전트 전용 |
| 세션 상태 자동감지 | ✓ (running/waiting/idle/error) | ✗ | ✓ (파란 링 + 사이드바) |
| MCP 토글 | ✓ (per-session, pool) | ✗ | △ (수동) |
| Skill 매니저 | ✓ | ✗ | △ |
| 포크 (대화 분기) | ✓ | ✗ | △ |
| 비용 추적 | ✓ | ✗ | ✗ |
| Conductor / Watcher | ✓ | ✗ | ✗ |
| 워크트리 통합 | ✓ | ✗ (수동 plugin) | △ |
| Docker 샌드박스 | ✓ | ✗ | ✗ |
| 리모트 SSH 통합 | ✓ | △ (수동) | ✓ |
| 학습곡선 | 중 | 높음 | 낮음 |

서로 배제하지 않음. agent-deck 이 내부에서 tmux 세션을 사용하므로 같이 도는 셈. cmux 는 macOS GUI 라 컨셉이 다름 — agent-deck 으로 헤드리스 / 리모트, cmux 로 로컬 GUI 식으로 나눠도 좋음.

---

## 비슷한 도구

agent-deck 이름이 워낙 흔해서 GitHub 에 동명 프로젝트가 여럿 있습니다. 이 문서가 가리키는 건 **`asheshgoplani/agent-deck`**:

- [claude-world/agent-deck](https://github.com/claude-world/agent-deck) — 웹 기반 멀티 에이전트 오케스트레이터 (DAG 실행)
- [tonyofthehills/agent-deck](https://github.com/tonyofthehills/agent-deck) — Mac 메뉴바 → 폰 알림
- [puritysb/AgentDeck](https://github.com/puritysb/AgentDeck) — Stream Deck+ / ESP32 같은 물리 컨트롤러
- [weykon/agent-deck](https://github.com/weykon/agent-deck), [starascendin/agent-deck-webui](https://github.com/starascendin/agent-deck-webui) — 포크 / 변형

같은 레이어의 도구:

- [mixpeek/amux](https://github.com/mixpeek/amux) — tmux 베이스 Claude Code 멀티플렉서, unattended 다중 에이전트
- [coder/mux](https://github.com/coder/mux) — 격리 병렬 에이전트 데스크톱 앱
- [cmux](./cmux.md) — macOS 네이티브 GUI 대안

---

## 라이선스

MIT © 2025 Ashesh Goplani.

---

## 참고

- 메인 repo: https://github.com/asheshgoplani/agent-deck
- 공식 사이트: https://asheshgoplani.github.io/agent-deck/
- README: https://github.com/asheshgoplani/agent-deck/blob/main/README.md
- Releases: https://github.com/asheshgoplani/agent-deck/releases
- 핸즈온 워크샵: [`../../workshops/extensions/agent-deck-workshop.md`](../../workshops/extensions/agent-deck-workshop.md)
- 관련 문서:
  - [tmux](./tmux.md) — 일반 멀티플렉서
  - [cmux](./cmux.md) — macOS GUI 대안
  - [Hooks](../tips/hooks.md) — 비용/알림 훅 박을 때
  - [MCP Setup](../tips/mcp-setup.md) — MCP 정의 기본
