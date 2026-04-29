# Remote 작업 환경 구성 워크샵

서버에 AI 에이전트를 띄워놓고, 외출 중에도 모바일로 확인/승인하는 환경을 처음부터 구성합니다.

> 사전 준비: 서버 또는 항상 켜둘 데스크톱 1대, 모바일 기기 (iOS 또는 Android)

---

## 1. 이 워크샵에서 만드는 것

완성 후 워크플로우:

```
서버에서 Claude Code 실행 → 외출 → 폰으로 ntfy 알림 수신
→ Termius로 SSH 접속 → tmux attach → 결과 확인/승인
→ (필요하면) Parsec/Stardesk로 GUI 확인
```

---

## 2. tmux 설치 & 세션 만들기

tmux가 없다면 먼저 설치:

```bash
# macOS
brew install tmux

# Ubuntu/Debian
sudo apt install tmux
```

세션을 만들고 Claude Code를 실행합니다:

```bash
# 세션 생성
tmux new -s claude

# 이 안에서 Claude Code 시작
claude
```

### 세션에서 빠져나오기 (detach)

```
Ctrl+b → d
```

프로세스는 그대로 돌아가고, 터미널만 빠져나옵니다.

### 다시 붙기 (attach)

```bash
tmux attach -t claude
```

> tmux 상세 사용법은 [tmux 문서](../../docs/terminal/tmux.md) 참고.

---

## 3. Tailscale 설치 — 어디서든 접속

공유기 뒤에 있는 서버에 외부에서 접속하려면 포트포워딩이 필요한데, Tailscale을 쓰면 그런 거 없이 바로 연결됩니다.

### 3-1. 서버에 설치

```bash
# macOS
brew install tailscale
sudo tailscale up

# Ubuntu/Debian
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

브라우저가 열리면 구글/GitHub 계정으로 로그인.

### 3-2. 모바일에도 설치

- iOS: App Store에서 "Tailscale" 검색 → 설치 → 같은 계정 로그인
- Android: Play Store에서 "Tailscale" 검색 → 설치 → 같은 계정 로그인

### 3-3. 연결 확인

```bash
tailscale status
```

서버와 모바일이 같은 네트워크에 보이면 성공. 이제 `100.x.x.x` IP 또는 MagicDNS 이름으로 접속 가능.

---

## 4. Termius 설치 — SSH 클라이언트

### 4-1. 모바일 앱 설치

- iOS: App Store에서 "Termius" 검색
- Android: Play Store에서 "Termius" 검색

### 4-2. 호스트 추가

1. Termius 열기 → `+` → New Host
2. 입력:
   - **Hostname**: Tailscale IP (`100.x.x.x`) 또는 MagicDNS 이름
   - **Username**: 서버 사용자명
   - **Password** 또는 **SSH Key** 설정
3. 저장 → 탭해서 접속

### 4-3. tmux 세션에 붙기

접속 후:

```bash
tmux attach -t claude
```

Claude Code가 돌아가는 화면이 그대로 보입니다.

---

## 5. 화면 보기 — 원격 데스크톱 (선택)

SSH + tmux만으로 충분하지만, 브라우저 확인이나 GUI 앱이 필요한 경우:

### Android — Parsec

1. 서버에 Parsec 설치: [parsec.app](https://parsec.app) → Download
2. 서버에서 Parsec 로그인 & Host 활성화
3. Android에서 Parsec 앱 설치 → 같은 계정 로그인
4. 서버 선택 → 연결

### iOS — Stardesk

1. 서버에 Stardesk Host 설치: [stardesk.net](https://www.stardesk.net/)
2. iOS에서 Stardesk 앱 설치
3. 페어링 코드로 연결

---

## 6. ntfy 알림 설정 — 핵심

에이전트가 끝났는데 모니터 앞에 없으면? → ntfy로 폰에 푸시.

### 6-1. 모바일 앱 설치

- iOS: App Store에서 "ntfy" 검색
- Android: Play Store에서 "ntfy" 검색

### 6-2. Topic 구독

앱 열기 → `+` → topic 이름 입력 (예: `my-claude-work`)

> topic 이름이 곧 채널. 남이 추측 못할 고유한 이름을 쓰세요.

### 6-3. 서버에서 테스트

```bash
curl -d "테스트 알림" ntfy.sh/my-claude-work
```

폰에 알림이 오면 성공.

### 6-4. Claude Code Hook 설정

`~/.claude/settings.json` 에 추가:

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "curl -s -d '작업 완료 — 확인 필요' ntfy.sh/my-claude-work"
          }
        ]
      }
    ],
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "curl -s -d '권한 승인 필요' ntfy.sh/my-claude-work"
          }
        ]
      }
    ]
  }
}
```

이제 Claude Code가 멈추거나 권한을 요청할 때마다 폰에 푸시가 옵니다.

### 6-5. 원격 승인/거부 (고급)

ntfy만으로는 알림만 받을 수 있고, 승인/거부까지 하려면 추가 도구가 필요합니다.

참고: [Reddit — Claude Code 원격 승인 도구](https://www.reddit.com/r/ClaudeAI/comments/1rbg82b/i_made_a_tool_to_approvedeny_claude_code/)

---

## 7. 전체 셋업 점검

모든 설정이 끝나면 이 체크리스트로 확인:

- [ ] 서버에 tmux 설치됨
- [ ] 서버 + 모바일에 Tailscale 설치, 같은 계정 로그인
- [ ] `tailscale status` 에서 양쪽 기기 보임
- [ ] Termius에서 Tailscale IP로 SSH 접속 성공
- [ ] SSH 접속 후 `tmux attach` 로 세션 복귀 가능
- [ ] ntfy 앱에서 topic 구독 완료
- [ ] `curl -d "test" ntfy.sh/my-topic` 으로 알림 수신 확인
- [ ] Claude Code hooks에 ntfy 설정 완료
- [ ] (선택) Parsec 또는 Stardesk 연결 테스트

---

## 8. 다음 단계

- tmux 심화: [tmux 문서](../../docs/terminal/tmux.md) — window/pane 분할, 커스텀 설정
- 알림 심화: [알림 가이드](../../docs/tips/notifications.md) — Slack, Discord, Telegram 연동
- 생산성: [생산성 팁](../../docs/tips/productivity.md) — 권한 설정, 세션 관리
- Remote 레퍼런스: [Remote 작업 환경](../../docs/tips/remote.md) — 각 도구별 상세 스펙
