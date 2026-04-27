# tmux

가장 표준적인 터미널 멀티플렉서. AI 코딩 워크플로우의 거의 필수 도구.

---

## 핵심 개념

- **session**: 작업 세트의 최상위 단위. 보통 프로젝트 단위로.
- **window**: 세션 안의 탭.
- **pane**: window 안의 분할 영역.

```
session
├── window 1
│   ├── pane (top)
│   └── pane (bottom)
└── window 2
    └── pane
```

세션은 **detach** 해도 백그라운드에서 살아있음. 다시 **attach** 하면 같은 상태로.

---

## 설치

```bash
# macOS
brew install tmux

# Ubuntu/Debian
sudo apt install tmux

# 확인
tmux -V
```

---

## 기본 단축키

기본 prefix는 `Ctrl+b`. 모든 단축키는 prefix 누른 뒤 다음 키.

### 세션

| 명령 | 동작 |
|---|---|
| `tmux new -s <name>` | 새 세션 시작 |
| `tmux ls` | 세션 목록 |
| `tmux a -t <name>` | 세션 attach |
| `prefix` `d` | detach (세션은 살아있음) |
| `prefix` `s` | 세션 선택 메뉴 |
| `prefix` `$` | 세션 이름 변경 |

### Window (탭)

| 명령 | 동작 |
|---|---|
| `prefix` `c` | 새 window |
| `prefix` `n` / `p` | 다음 / 이전 window |
| `prefix` `<숫자>` | 해당 번호 window로 |
| `prefix` `,` | window 이름 변경 |
| `prefix` `&` | window 닫기 |

### Pane (분할)

| 명령 | 동작 |
|---|---|
| `prefix` `%` | 좌우 분할 |
| `prefix` `"` | 상하 분할 |
| `prefix` `↑↓←→` | pane 간 이동 |
| `prefix` `z` | 현재 pane 줌 (toggle) |
| `prefix` `x` | pane 닫기 |
| `prefix` `{` / `}` | pane 자리 바꾸기 |
| `prefix` `space` | 레이아웃 순환 |

### 복사 모드

| 명령 | 동작 |
|---|---|
| `prefix` `[` | 복사 모드 진입 (스크롤 가능) |
| `q` | 복사 모드 종료 |
| `space` | 선택 시작 (복사 모드 안에서) |
| `enter` | 선택 복사 |
| `prefix` `]` | 붙여넣기 |

---

## 추천 `~/.tmux.conf`

```bash
# prefix를 Ctrl+a 로 (Ctrl+b 보단 누르기 쉬움)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# 마우스 활성 (스크롤/리사이즈)
set -g mouse on

# 분할 단축키 직관적으로
bind | split-window -h
bind - split-window -v

# 인덱스 1부터 (키패드 누르기 편하게)
set -g base-index 1
setw -g pane-base-index 1

# 24bit 컬러
set -g default-terminal "tmux-256color"
set -ga terminal-overrides ",xterm-256color:Tc"

# 히스토리 늘리기
set -g history-limit 50000

# 상태바 색상 / 정보
set -g status-bg colour234
set -g status-fg colour137
set -g status-right '#[fg=colour233,bg=colour241,bold] %Y-%m-%d #[fg=colour233,bg=colour245,bold] %H:%M '

# vi-style 복사 모드
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send -X copy-pipe-and-cancel "pbcopy"  # macOS

# 설정 리로드
bind r source-file ~/.tmux.conf \; display "Reloaded!"
```

설정 변경 후 `prefix` `r` 또는 `tmux source-file ~/.tmux.conf`.

---

## AI 코딩 친화적 레이아웃

### 패턴 1: 에이전트 + 로그

```
prefix |       # 좌우 분할
좌측: claude
우측: tail -f logs / pnpm test --watch
```

### 패턴 2: 메인 + 서브

```
prefix -       # 상하 분할
prefix |       # 위쪽을 다시 좌우로
좌상: editor (nvim 또는 그냥 비움)
우상: claude
하단: 셸 / git / 로그
```

### 패턴 3: 멀티 에이전트

```
세션 하나, 4개 pane
1: claude (메인 에이전트)
2: aider --watch-files (자동 보조)
3: 셸 (테스트/빌드)
4: 로그 tail
```

---

## 추천 플러그인 (TPM)

```bash
# TPM (Tmux Plugin Manager)
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

`~/.tmux.conf` 끝에:

```bash
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-resurrect'   # 세션 저장/복원
set -g @plugin 'tmux-plugins/tmux-continuum'   # 자동 저장
set -g @plugin 'tmux-plugins/tmux-yank'        # 클립보드 통합
run '~/.tmux/plugins/tpm/tpm'
```

설치: `prefix` `I`. 업데이트: `prefix` `U`.

`tmux-resurrect`는 노트북 재부팅 후에도 세션 복원해줘서 매우 유용.

---

## 자주 쓰는 워크플로우 명령어

```bash
# 프로젝트별 세션 정형화
tmux new -d -s myproject -c ~/work/myproject
tmux send-keys -t myproject "claude" Enter

# 한 번에 띄우기 (스크립트화)
cat > ~/bin/work-myproject.sh <<'EOF'
#!/bin/bash
SESSION=myproject
tmux has-session -t $SESSION 2>/dev/null
if [ $? != 0 ]; then
  tmux new -d -s $SESSION -c ~/work/myproject
  tmux rename-window -t $SESSION:0 'main'
  tmux send-keys -t $SESSION:main 'claude' Enter
  tmux new-window -t $SESSION -n 'logs'
  tmux send-keys -t $SESSION:logs 'pnpm dev' Enter
fi
tmux attach -t $SESSION
EOF
chmod +x ~/bin/work-myproject.sh
```

`tmuxinator` / `tmuxp` 로 더 깔끔하게 가능.

---

## 흔한 트러블슈팅

### 색이 이상하다

```bash
# ~/.zshrc 또는 ~/.bashrc
alias tmux="TERM=xterm-256color tmux"
```

또는 `.tmux.conf` 의 `default-terminal` 설정 확인.

### macOS에서 클립보드 연동 안 됨

`reattach-to-user-namespace` 가 옛날엔 필요했지만, 최근 macOS + tmux 3.x 에선 보통 그냥 됩니다. 안 되면:

```bash
brew install reattach-to-user-namespace
# .tmux.conf 에 set-option -g default-command "reattach-to-user-namespace -l zsh"
```

### prefix 가 자꾸 안 먹는다

Ctrl+a 로 바꿨다면 readline의 라인 시작 단축키와 충돌함. 한 번씩 빠지는 정도는 감수하거나 다시 Ctrl+b 로.

---

## 참고

- 공식: https://github.com/tmux/tmux
- 치트시트: https://tmuxcheatsheet.com/
- 책: "tmux 2: Productive Mouse-Free Development" (Brian P. Hogan)
