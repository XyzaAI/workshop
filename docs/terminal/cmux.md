# cmux

[manaflow-ai/cmux](https://github.com/manaflow-ai/cmux) — **AI 코딩 에이전트를 위해 만든 macOS 네이티브 터미널**. Ghostty(libghostty) 기반, GPU 가속.

> tmux 류의 "터미널 멀티플렉서" 가 아니라 **GUI 앱**. 일반 멀티플렉서 카테고리보단 "AI 에이전트 전용 IDE-급 셸 환경" 이라 봐야 적절합니다.

---

## 왜 만들어졌나

여러 AI 에이전트를 동시에 굴릴 때 일반 터미널은:
- 어떤 pane이 입력 기다리는지 즉시 안 보임
- 알림이 OS 레벨로 안 옴
- worktree / 브랜치별 컨텍스트가 헷갈림
- 브라우저 자동화 따로 띄우기 번거로움

cmux는 이걸 한 앱에 다 넣음.

---

## 설치

### Homebrew (cask)

```bash
brew tap manaflow-ai/cmux
brew install --cask cmux
```

### DMG

[Releases 페이지](https://github.com/manaflow-ai/cmux/releases) 에서 다운로드 → Applications에 드래그. Sparkle로 자동 업데이트.

**macOS 전용**.

---

## 핵심 기능

### 1. 사이드바 — 작업 한눈에

각 워크스페이스마다 다음 정보 표시:
- 현재 git 브랜치
- 연결된 PR 상태/번호
- 작업 디렉토리
- listen 중인 포트
- **최근 알림 텍스트**

여러 프로젝트 / worktree를 동시에 굴려도 어디서 무슨 일 일어나는지 즉시 파악.

### 2. 알림 (이게 진짜 강점)

- 에이전트가 입력 기다리면 **pane에 파란 링** + 사이드바에 알림
- OSC 9 / 99 / 777 시퀀스를 알림으로 변환
- CLI: `cmux notify "메시지"` — Claude Code / OpenCode 훅에 그대로 박을 수 있음

```json
// 예: ~/.claude/settings.json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "cmux notify '작업 완료'" }]
      }
    ]
  }
}
```

### 3. 내장 브라우저

스크립트 가능한 브라우저 pane. agent-browser API 포팅:
- Accessibility tree 스냅샷
- 요소 ref / 클릭 / 폼 채우기
- JS evaluate
- 에이전트가 직접 조작

### 4. 수직 + 수평 탭

```
+------+--------------------+--------------------+
| 사이드바 | claude (메인)    | codex (서브)        |
| - p1   |                  |                    |
| - p2   +------------------+--------------------+
| - p3   | 셸 / 로그         | 브라우저            |
+--------+------------------+--------------------+
```

### 5. SSH 워크스페이스

리모트 머신을 워크스페이스로 등록. 브라우저는 리모트 네트워크로 라우팅.

### 6. Claude Code 팀(Teammate) 모드 네이티브

여러 Claude Code 인스턴스를 팀처럼 묶어서 작업 분배.

### 7. 세션 복원

레이아웃, 디렉토리, 스크롤백, 브라우저 히스토리 저장 → 재시작 후 그대로.

---

## 호환 에이전트

명시적 지원: **Claude Code, Codex**.
알림 시스템(OSC 시퀀스 + `cmux notify` CLI)은 어떤 에이전트든 훅만 박으면 동작 — OpenCode, Gemini CLI, Aider, Kiro 등 다 가능.

---

## tmux 와 비교

| 항목 | cmux | tmux |
|---|---|---|
| 형태 | macOS GUI 앱 | CLI |
| 플랫폼 | macOS only | 모든 \*nix |
| AI 에이전트 친화 | 매우 강함 (네이티브 알림/브라우저) | 직접 셋업 |
| 학습곡선 | 낮음 (보통 GUI) | 높음 (단축키/세션) |
| SSH/리모트 | 내장 | 직접 |
| 가벼움 | 무거움 | 매우 가벼움 |

병행도 가능 — cmux 안에서 ssh 로 리눅스 들어가서 tmux 쓰는 식.

---

## 비슷한 도구

- **[mixpeek/amux](https://github.com/mixpeek/amux)** — 오픈소스 Claude Code 멀티플렉서, **tmux 베이스**. 수십 개 병렬 에이전트를 unattended 로. CLI 환경(Linux/macOS) 친화. cmux보다 가벼움.
- **[coder/mux](https://github.com/coder/mux)** — 격리된 병렬 에이전트 개발용 데스크톱 앱. 다른 컨셉.
- **[wmux](https://github.com/openwong2kim/wmux)** — Windows 포팅. 같은 design / socket 프로토콜.
- **[boundsj/agent-skills - cmux skill](https://github.com/boundsj/agent-skills/blob/main/cmux/SKILL.md)** — Claude Code용 cmux 활용 스킬.
- **[hummer98/using-cmux](https://github.com/hummer98/using-cmux)** — Claude Code용 cmux 스킬.

---

## 라이선스

GPL-3.0-or-later (상용 라이선스 별도 협의 가능).

---

## 참고

- 메인: https://github.com/manaflow-ai/cmux
- 공식 사이트: https://cmux.com
- Claude Hub 리뷰: https://www.claude-hub.com/resource/github-cli-manaflow-ai-cmux-cmux/
