# 알림 / Notifications

AI 에이전트는 작업이 30초~몇 분씩 걸리는 일이 많습니다. **언제 끝났는지** 정확히 알아야 멀티태스킹이 가능합니다.

이 문서는 Claude Code 기준이지만, 거의 모든 에이전트에 응용 가능.

---

## 1. macOS 시스템 알림 (가장 기본)

### Hook으로 작업 완료 알림

`~/.claude/settings.json` :

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude 작업 완료\" with title \"Claude Code\" sound name \"Glass\"'"
          }
        ]
      }
    ]
  }
}
```

`Stop` 이벤트는 에이전트가 사용자 입력을 기다리는 시점마다 발동.

### 권한 요청 알림 (다른 일 하다가 묻혀 놓치는 경우)

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"권한/입력 필요\" with title \"Claude Code\" sound name \"Tink\"'"
          }
        ]
      }
    ]
  }
}
```

---

## 2. 소리 / 음성

### 작업 완료 시 짧은 효과음

```bash
# afplay 는 macOS 기본
afplay /System/Library/Sounds/Glass.aiff
```

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Glass.aiff" }]
    }]
  }
}
```

### TTS로 결과 요약 읽어주기

```bash
# 한국어 음성 (macOS 내장)
say -v Yuna "작업 완료"

# 영어
say "Done"
```

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{ "type": "command", "command": "say -v Yuna '끝났어요'" }]
    }]
  }
}
```

> 회의 중일 때 자동 음소거 토글하는 더 똑똑한 버전도 가능 (focus mode 감지).

---

## 3. 메신저 알림

### Slack DM으로 알림

slack CLI 또는 webhook 사용:

```bash
# webhook
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Claude 작업 완료"}' \
  https://hooks.slack.com/services/XXX/YYY/ZZZ
```

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "curl -s -X POST -H 'Content-type: application/json' --data '{\"text\":\"✅ Claude 작업 완료\"}' $SLACK_WEBHOOK_URL"
      }]
    }]
  }
}
```

`$SLACK_WEBHOOK_URL` 은 `.env` 또는 셸 환경변수에 두기.

### Discord webhook

```bash
curl -H "Content-Type: application/json" \
  -d '{"content":"✅ 작업 완료"}' \
  $DISCORD_WEBHOOK_URL
```

### Telegram bot

```bash
curl -X POST "https://api.telegram.org/bot$TG_TOKEN/sendMessage" \
  -d "chat_id=$TG_CHAT_ID" \
  -d "text=✅ 작업 완료"
```

---

## 4. 모바일 푸시 (자리 비울 때 유용)

### Pushover

```bash
curl -s \
  --form-string "token=$PUSHOVER_APP_TOKEN" \
  --form-string "user=$PUSHOVER_USER_KEY" \
  --form-string "message=Claude 작업 완료" \
  https://api.pushover.net/1/messages.json
```

### ntfy (무료, 셀프호스팅 가능)

```bash
curl -d "Claude 작업 완료" ntfy.sh/your-unique-topic
```

모바일 앱(`ntfy`)을 깔고 같은 topic 구독하면 끝. 가장 가볍게 시작 가능.

### Apple Push (자동화 앱 활용)

iPhone 단축어 + Webhook 받는 자동화 만들면 iMessage/Push 연동 가능.

---

## 5. 컨텍스트 별로 다른 알림

작업 종류에 따라 다르게 알리기:

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [{
          "type": "command",
          "command": "if grep -q 'error\\|failed' $CLAUDE_LAST_OUTPUT 2>/dev/null; then say -v Yuna '에러 났어요'; else say -v Yuna '완료'; fi"
        }]
      }
    ]
  }
}
```

> 정확한 환경변수명/이벤트명은 Claude Code 버전에 따라 다를 수 있음. `/help` 또는 공식 문서 확인.

---

## 6. 추천 셋업 (개인적으론)

가벼운 데일리 드라이버용:

1. **로컬 작업 중** → `osascript display notification` + 짧은 효과음 (Glass.aiff)
2. **회의 중 / 자리 비움** → ntfy 푸시
3. **에러 발생 시만** → TTS로 음성 알림 (`say`)
4. **중요한 PR 리뷰 끝났을 때** → Slack DM

너무 많이 알리면 무시하게 되니 **이벤트 종류 + 중요도**로 구분하는 게 핵심.

---

## 7. 하지 말아야 할 것

- **모든 hook에 시끄러운 알림** — 곧 알림 자체를 무시하게 됨
- **민감 정보를 webhook으로** — 프롬프트/코드가 외부 서비스에 평문으로 가지 않게
- **무한 루프** — hook 안에서 다시 claude를 호출하는 식의 구성

---

## 참고

- macOS 알림 옵션: `osascript -e 'display notification ...'` 의 sound, subtitle 등
- 이벤트 종류: Claude Code의 `Stop`, `Notification`, `PreToolUse`, `PostToolUse`, `UserPromptSubmit`
- Hooks 자세히: [`./hooks.md`](./hooks.md)
