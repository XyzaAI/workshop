# 터미널 / 멀티플렉서

AI 코딩 에이전트는 **터미널 안에서 굴러갑니다**. 그래서 터미널 환경 자체를 잘 잡아두는 것이 생산성 차이를 크게 만듭니다.

---

## 왜 멀티플렉서를 써야 하나

AI 에이전트는 다음 같은 워크플로우가 일상입니다:

- 한쪽에 에이전트 띄워놓고, 다른 쪽에서 로그/테스트/git 보기
- 여러 프로젝트의 에이전트 동시에
- SSH 연결이 끊겨도 작업 유지
- 노트북 닫고 다른 머신에서 같은 세션 이어서

이걸 **멀티플렉서** 없이는 불가능에 가깝습니다.

---

## 도구 비교

| 도구 | 한 줄 정리 | 추천 상황 |
|---|---|---|
| [tmux](./tmux.md) | 표준 멀티플렉서, 학습 곡선 있지만 가장 강력 | 데일리 드라이버, SSH 작업 |
| [cmux](./cmux.md) | Claude Code/AI 친화적 wrapper 또는 대안 | AI 작업 전용 환경 |
| zellij | tmux 대체, 친절한 UI | tmux 단축키가 너무 부담스러울 때 |
| screen | 기본 내장 | 굳이 새로 배울 가치 적음 |

---

## 추천 셋업 (개인 차이 큼)

### 단순 셋업

```
tmux 세션 1개
├── window 1: editor (VSCode/nvim)
├── window 2: claude
└── window 3: tests/server logs
```

### 복합 셋업

```
프로젝트마다 tmux 세션 분리
├── session: project-a
│   ├── window 1: editor
│   ├── window 2: claude
│   └── window 3: server
└── session: project-b
    ├── ...
```

`tmuxinator` / `tmux-resurrect` 같은 플러그인으로 셋업 자동 복원.

### AI 멀티에이전트

```
한 화면에 여러 에이전트 동시:
- pane 1: claude code (메인)
- pane 2: codex (서브)
- pane 3: aider --watch-files (자동 모드)
- pane 4: 로그 / git status
```

---

## 부가 도구

- **mosh** — SSH 대체. 끊어도 자동 재연결, 모바일/불안정 네트워크에 좋음
- **lazygit** / **gh** — 터미널 안에서 git/GitHub 다루기
- **fzf** / **ripgrep** — 빠른 검색, 에이전트도 사용
- **bat** — `cat` 대체, 색상 있어 가독성 좋음
