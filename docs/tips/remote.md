# Remote 작업 환경

서버에 AI 에이전트를 띄워두고, 어디서든 접속해서 작업하는 환경 구성 가이드.

> 핸즈온 가이드는 [`../../workshops/tips/remote-workshop.md`](../../workshops/tips/remote-workshop.md) 참고. 이 문서는 **레퍼런스 / 심화** 위주.

---

## 왜 Remote인가

- AI 에이전트는 작업 시간이 길다 — 노트북 닫아도 서버에서 계속 돌아야 함
- tmux + SSH 조합이면 어디서든 세션 복귀 가능
- iPad/폰에서도 코드 리뷰, 승인, 모니터링 가능

---

## 구성 요소 한눈에

| 레이어 | 도구 | 역할 |
|---|---|---|
| 세션 유지 | **tmux** | SSH 끊어져도 프로세스 유지 |
| SSH 접속 | **Termius** | 크로스플랫폼 SSH 클라이언트 |
| 네트워크 | **Tailscale** | 공유기/방화벽 뒤에서도 P2P 접속 |
| 화면 보기 (Android) | **Parsec** | 저지연 원격 데스크톱 |
| 화면 보기 (iOS) | **Stardesk** | iOS 네이티브 원격 데스크톱 |
| 알림 | **ntfy** | 모바일 푸시 알림 (무료, 셀프호스팅 가능) |

---

## 1. tmux — 세션 유지

자세한 사용법은 [`../terminal/tmux.md`](../terminal/tmux.md) 참고.

Remote 환경에서 핵심인 이유:

- SSH 연결이 끊어져도 tmux 세션 안의 프로세스는 계속 실행
- `tmux attach` 로 언제든 복귀
- Claude Code 같은 인터랙티브 에이전트를 띄워놓고 외출 가능

```bash
# 세션 생성
tmux new -s work

# 세션에서 빠져나오기 (프로세스는 유지)
# Ctrl+b → d

# 다시 붙기
tmux attach -t work
```

---

## 2. SSH — Termius + Tailscale

### Termius

크로스플랫폼 SSH 클라이언트. macOS, Windows, iOS, Android 모두 지원.

- **SFTP 내장** — 파일 전송도 같은 앱에서
- **Snippets** — 자주 쓰는 명령어 저장
- **Port Forwarding** — GUI로 편하게 설정
- **Vault** — SSH 키/비밀번호 암호화 저장, 기기 간 동기화

> 무료 플랜으로도 기본 SSH 접속 충분. 키 동기화/SFTP는 프리미엄.

공식 사이트: [termius.com](https://termius.com)

### Tailscale

WireGuard 기반 메시 VPN. 공유기 뒤에 있는 서버에도 직접 접속 가능하게 해줌.

- **Zero config** — 설치하고 로그인하면 끝
- **NAT 통과** — 포트포워딩, DDNS 불필요
- **MagicDNS** — `macstudio.tail1234.ts.net` 같은 도메인으로 접속
- **Exit Node** — 특정 기기를 VPN 게이트웨이로 사용 가능
- **무료 플랜** — 개인 사용 100대까지 무료

```bash
# 설치 (macOS)
brew install tailscale

# 시작
sudo tailscale up

# 상태 확인
tailscale status
```

**조합**: Tailscale로 네트워크 연결 → Termius로 SSH 접속 → tmux 세션에 attach

---

## 3. 화면 보기 — 원격 데스크톱

SSH만으로 충분하지 않을 때 (GUI 앱, 브라우저 확인 등).

### Android — Parsec

게이밍용 저지연 원격 데스크톱. 코딩에도 충분히 빠름.

- **저지연** — 게이밍 수준 (~15ms)
- **하드웨어 인코딩** — GPU 활용, CPU 부담 적음
- **4K/60fps** — 해상도 제한 없음
- **무료** — 개인 사용 무료

공식 사이트: [parsec.app](https://parsec.app)

### iOS — Stardesk

iOS 네이티브 원격 데스크톱 앱.

- macOS/Windows 호스트 지원
- 터치 최적화 인터페이스
- 저지연 스트리밍

공식 사이트: [stardesk.net](https://www.stardesk.net/)

---

## 4. 알림 — ntfy

서버에서 돌아가는 에이전트가 끝났을 때 모바일로 푸시 받기.

자세한 알림 옵션은 [`./notifications.md`](./notifications.md) 참고.

### ntfy 기본

```bash
# 서버에서 알림 보내기 (이게 전부)
curl -d "Claude 작업 완료" ntfy.sh/my-unique-topic
```

모바일에서 ntfy 앱 설치 → 같은 topic 구독 → 끝.

### Claude Code Hook 연동

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "curl -s -d '작업 완료' ntfy.sh/my-claude-topic"
      }]
    }]
  }
}
```

### 승인/거부 원격 제어

ntfy의 action 버튼을 활용하면 모바일에서 권한 승인/거부도 가능.

참고: [Reddit — Claude Code 원격 승인 도구](https://www.reddit.com/r/ClaudeAI/comments/1rbg82b/i_made_a_tool_to_approvedeny_claude_code/)

---

## 5. 추천 조합

### 기본 셋업

```
로컬 PC/서버
├── tmux 세션 안에서 Claude Code 실행
├── Tailscale 설치 (네트워크)
└── ntfy hook 설정 (알림)

모바일/태블릿
├── Termius (SSH → tmux attach)
├── Tailscale (네트워크)
├── ntfy 앱 (알림 수신)
└── Parsec 또는 Stardesk (GUI 필요 시)
```

### 워크플로우

1. 서버에서 `tmux new -s claude` → Claude Code 시작
2. 작업 지시 후 `Ctrl+b d` 로 detach (또는 그냥 SSH 끊기)
3. ntfy 알림이 오면 Termius로 SSH 접속 → `tmux attach -t claude`
4. 결과 확인, 추가 지시 또는 승인
5. GUI 확인 필요하면 Parsec/Stardesk로 화면 보기

---

## 참고 링크

- tmux 상세: [`../terminal/tmux.md`](../terminal/tmux.md)
- 알림 설정: [`./notifications.md`](./notifications.md)
- Termius: [termius.com](https://termius.com)
- Tailscale: [tailscale.com](https://tailscale.com)
- Parsec: [parsec.app](https://parsec.app)
- Stardesk: [stardesk.net](https://www.stardesk.net/)
- ntfy: [ntfy.sh](https://ntfy.sh)
