# Hooks

Claude Code (및 일부 다른 에이전트)에서 **특정 이벤트마다 셸 명령을 자동 실행**하는 훅 시스템. 자동화의 핵심.

---

## 무엇을 할 수 있나

- 파일 편집할 때마다 lint/format 자동 실행
- bash 명령 실행 전에 검증
- 작업 끝나면 알림
- 권한 요청 시 텍스트 음성 변환
- "위험한 명령" 차단

---

## 이벤트 종류

| 이벤트 | 발동 시점 |
|---|---|
| `UserPromptSubmit` | 사용자가 프롬프트 입력 직후 |
| `PreToolUse` | 도구 호출 직전 (차단 가능) |
| `PostToolUse` | 도구 호출 직후 |
| `Notification` | 권한 요청 등 사용자 입력 대기 |
| `Stop` | 에이전트 응답 완료 |
| `SessionStart` / `SessionEnd` | 세션 시작/종료 |
| `SubagentStop` | 서브에이전트 종료 |
| `PreCompact` | 컨텍스트 압축 직전 |

(정확한 목록은 Claude Code 버전마다 다름. `/help` 또는 공식 문서 확인.)

---

## 기본 형태

`~/.claude/settings.json` 또는 `<project>/.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "pnpm lint --fix"
          }
        ]
      }
    ]
  }
}
```

`matcher`: 어떤 도구일 때 발동할지 정규식 (생략하면 모두).
`type`: 보통 `command`.
`command`: 실행할 셸 명령.

---

## 자주 쓰는 패턴

### 1. 편집 후 자동 lint/format

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [{ "type": "command", "command": "pnpm prettier --write $CLAUDE_FILE_PATHS" }]
      }
    ]
  }
}
```

> 환경변수 (`$CLAUDE_FILE_PATHS` 등) 이름은 버전마다 다를 수 있음. 공식 문서 확인.

### 2. 위험한 bash 차단

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -E 'rm -rf /|sudo|curl.*\\|.*sh'; then echo '차단됨' >&2; exit 1; fi"
        }]
      }
    ]
  }
}
```

`exit 1` 으로 도구 호출을 차단. `stderr` 메시지는 사용자에게 표시.

### 3. 작업 완료 알림 (notifications.md 참조)

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Glass.aiff" }]
      }
    ]
  }
}
```

### 4. 세션 시작 시 컨텍스트 주입

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [{
          "type": "command",
          "command": "echo \"오늘 날짜: $(date '+%Y-%m-%d %A')\""
        }]
      }
    ]
  }
}
```

stdout이 컨텍스트에 추가됨 → 에이전트가 인지.

### 5. 매 PR 작업 시작 전 git fetch

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "git fetch --all --prune" }]
      }
    ]
  }
}
```

### 6. 프롬프트 자동 확장

`UserPromptSubmit` 에서 사용자 프롬프트를 가공:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [{
          "type": "command",
          "command": "echo '\\n[자동 추가] 변경 시 반드시 테스트도 같이 작성해줘.'"
        }]
      }
    ]
  }
}
```

---

## 글로벌 vs 프로젝트

- **`~/.claude/settings.json`** — 모든 프로젝트에 적용. 알림, 음성 같은 개인 환경.
- **`<project>/.claude/settings.json`** — 프로젝트 한정. 팀과 공유할 lint/test 훅. 커밋 대상.
- **`<project>/.claude/settings.local.json`** — 개인용 프로젝트 설정. `.gitignore` 추천.

같은 이벤트에 여러 hook이 정의되면 모두 실행.

---

## 디버깅

훅이 의도대로 안 도는 경우:

1. **먼저 셸에서 직접 실행** — 명령 자체가 동작하는지
2. **로그 남기기** — `command` 끝에 `>> ~/claude-hook.log 2>&1` 붙여서 출력 캡처
3. **환경변수 확인** — Claude Code가 넘기는 변수명을 명확히 하고 사용
4. **권한 모드 확인** — Bash hook이 도는데 도구가 거절되는 경우 `permissions` 충돌

---

## 위험 / 주의

- Hook 명령은 **사용자 권한**으로 실행됨. 신중히.
- `PreToolUse` 에서 무거운 명령 → 매 호출 지연됨
- 외부 네트워크 요청 → 토큰/지연/요금 영향
- 무한 루프 — hook 안에서 다시 도구 호출 트리거하지 않게 주의

---

## 다른 에이전트에서

- **OpenCode**: hook 시스템 비슷한 형태로 제공 (config.json 안에)
- **Aider**: 직접적인 hook은 없지만 `lint-cmd` / `test-cmd` 옵션이 비슷한 역할
- **Crush**: 개발 중 / 제한적
- **Codex**: sandbox 정책이 비슷한 효과

---

## 참고

- Claude Code 공식 hooks 문서
- 알림 응용: [`./notifications.md`](./notifications.md)
- 권한과 관계: 본 폴더 [`./productivity.md`](./productivity.md) 의 권한 섹션
