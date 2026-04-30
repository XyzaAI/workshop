# Claude Code Channels 핸즈온 워크샵

[Channels](https://code.claude.com/docs/en/channels) 는 **외부 시스템(채팅 / 웹훅 / CI / 모니터링)에서 발생한 이벤트를 이미 실행 중인 Claude Code 세션 안으로 push** 하는 MCP 기반 확장입니다.

기존 통합과의 차이:

```
Claude Code on the Web   : 새 클라우드 세션 (clone & run)
Claude in Slack          : @멘션으로 새 web 세션 spawn
표준 MCP 서버            : Claude 가 필요할 때 pull
Remote Control           : 폰/웹에서 내 로컬 세션 조종
Channels                 : 외부 이벤트를 내 로컬 세션에 push  ← 빈자리 채움
```

> 한 줄 요약: **터미널을 열어둔 채로 자리 비울 때, 빌드 실패/메시지/알람을 폰으로 받아 바로 Claude 가 움직이게** 하고 싶을 때 쓰는 기능.

이 워크샵에서:

- `fakechat` 데모로 채널 동작 원리 체감
- Telegram / Discord / iMessage 연결
- Permission Relay 로 폰에서 권한 승인
- 직접 채널 만들기 (webhook 수신기)

---

## 0. 사전 준비

| 항목 | 확인 |
|---|---|
| Claude Code v2.1.80 이상 | `claude --version` |
| **claude.ai 로그인 인증** | API Key / Console 인증은 **미지원** |
| [Bun](https://bun.sh) 설치 | `bun --version` |
| Team / Enterprise 사용자 | 관리자가 `channelsEnabled: true` 켜야 함 (Pro/Max 는 기본 사용 가능) |

> Permission Relay 는 v2.1.81+ 필요.

> Research Preview 단계라 `--channels` 플래그 / 프로토콜이 변경될 수 있음.

---

## 1. Quickstart — fakechat 으로 동작 확인

`fakechat` 은 localhost 에서 동작하는 공식 데모 채널입니다. 토큰/외부 서비스 없이 동작 확인용.

### 1-1. 플러그인 설치

```
> /plugin install fakechat@claude-plugins-official
```

> 마켓플레이스를 못 찾는다면:
> `/plugin marketplace update claude-plugins-official` 또는
> `/plugin marketplace add anthropics/claude-plugins-official` 후 재시도

### 1-2. 채널 활성화하면서 재시작

Claude Code 종료 후, `--channels` 와 함께 다시 시작:

```bash
claude --channels plugin:fakechat@claude-plugins-official
```

> `--channels` 에 여러 플러그인을 공백 구분해 넘길 수 있음.

### 1-3. 메시지 push 해보기

브라우저에서 [http://localhost:8787](http://localhost:8787) 열고 입력:

```
hey, what's in my working directory?
```

세션에 다음 형태로 도착:

```
<channel source="fakechat">hey, what's in my working directory?</channel>
```

Claude 가 작업 후 fakechat 의 `reply` 툴로 응답 → 브라우저 채팅 UI 에 답이 뜸.

> 터미널에는 **inbound 메시지는 보이지만 reply 텍스트는 안 보임**. 도구 호출 + "sent" 확정만 표시되고 실제 답은 외부 플랫폼 쪽에 표시됨.

---

## 2. Telegram 연결

### 2-1. 봇 만들기

1. Telegram 에서 [@BotFather](https://t.me/BotFather) 열기
2. `/newbot` 보내고 이름 / `…bot` 으로 끝나는 username 입력
3. BotFather 가 주는 **토큰 복사**

### 2-2. 플러그인 설치 + 토큰 등록

```
> /plugin install telegram@claude-plugins-official
> /reload-plugins                        # configure 커맨드 활성화
> /telegram:configure <token>            # ~/.claude/channels/telegram/.env 에 저장
```

> 또는 셸에서 `TELEGRAM_BOT_TOKEN=...` 환경변수로 띄워도 됨.

### 2-3. 채널 활성화 후 재시작

```bash
claude --channels plugin:telegram@claude-plugins-official
```

### 2-4. 페어링 (allowlist 등록)

1. Telegram 에서 봇에게 **아무 메시지나** 전송
2. 봇이 **페어링 코드** 회신
3. Claude Code 세션에서:

```
> /telegram:access pair <code>
> /telegram:access policy allowlist     # 본인만 허용
```

> 봇이 답이 없으면 → Claude Code 가 위 `--channels` 로 떠 있는지 확인. 채널이 떠 있어야 봇이 작동함.

---

## 3. Discord 연결

### 3-1. 봇 만들기

1. [Discord Developer Portal](https://discord.com/developers/applications) → **New Application**
2. **Bot** 섹션 → username 만들고 **Reset Token** → 토큰 복사
3. **Privileged Gateway Intents** → **Message Content Intent** 켜기
4. **OAuth2 → URL Generator** → scope `bot` + 다음 권한:
   - View Channels
   - Send Messages
   - Send Messages in Threads
   - Read Message History
   - Attach Files
   - Add Reactions
5. 생성된 URL 로 봇을 서버에 초대

### 3-2. 설치 / 설정 / 재시작

```
> /plugin install discord@claude-plugins-official
> /reload-plugins
> /discord:configure <token>
```

```bash
claude --channels plugin:discord@claude-plugins-official
```

### 3-3. 페어링

```
> /discord:access pair <code>
> /discord:access policy allowlist
```

본인 DM → 페어링 코드 → 등록.

---

## 4. iMessage 연결 (macOS 전용)

봇 토큰 / 외부 서비스 없이 macOS Messages DB 직접 읽고 AppleScript 로 회신합니다.

### 4-1. Full Disk Access 허가

`~/Library/Messages/chat.db` 가 macOS 보호 대상이라 첫 실행 시 권한 프롬프트가 뜸. **Allow** 클릭.

수동으로는 **System Settings → Privacy & Security → Full Disk Access** 에서 사용하는 터미널(Terminal/iTerm/IDE) 추가. 없으면 `authorization denied` 로 즉시 종료.

### 4-2. 설치 + 재시작

```
> /plugin install imessage@claude-plugins-official
```

```bash
claude --channels plugin:imessage@claude-plugins-official
```

### 4-3. 본인에게 메시지 보내기

같은 Apple ID 디바이스에서 자신에게 메시지 → **자기 자신은 access control 자동 우회**.

> 첫 회신 때 macOS Automation 권한 프롬프트(터미널이 Messages 제어해도 됩니까?) → **OK**.

### 4-4. 다른 사람 허용

```
> /imessage:access allow +821012345678
> /imessage:access allow user@example.com
```

---

## 5. Permission Relay — 폰에서 권한 승인

Claude 가 `Bash` / `Write` / `Edit` 같은 승인 필요 도구를 호출하면 → 로컬 터미널에 다이얼로그 + **2-way 채널이 capability 선언했다면 같은 프롬프트가 채널로도 전달**됨. 둘 중 먼저 답한 쪽이 적용.

> 프로젝트 trust / MCP 서버 동의는 relay 안 됨 (로컬 다이얼로그만).

### 사용자 입장 흐름

```
1. Claude 가 Bash 호출 시도
2. 터미널에 "Allow ?" 다이얼로그 + 채널로 메시지:
   "Claude wants to run Bash: list files
    Reply 'yes abcde' or 'no abcde'"
3. 폰에서 "yes abcde" 회신
4. 로컬 다이얼로그 자동 닫힘 + 도구 실행
```

ID 는 5글자 lowercase, `l` 제외 (폰에서 `1`/`I` 와 헷갈리지 않게).

> Telegram / Discord / iMessage 공식 플러그인은 이미 지원. 자신의 채널에 추가하는 법은 [§7](#7-내-채널-만들기-webhook-receiver) 참고.

unattended 환경에서 권한 프롬프트 자체를 우회하려면 [`--dangerously-skip-permissions`](https://code.claude.com/docs/en/permission-modes) — 신뢰하는 환경에서만.

---

## 6. 보안 — Sender Allowlist 와 Prompt Injection

채널이 외부 메시지를 Claude 에게 직접 push 하므로 **누가 보낼 수 있는지 게이트 필수**. 미설정이면 prompt injection 통로.

| 채널 | 게이트 방식 |
|---|---|
| Telegram / Discord | 페어링 → sender ID allowlist |
| iMessage | 본인 핸들 자동 허용, 타인은 `/imessage:access allow` |
| 자작 채널 | 직접 sender allowlist 구현 (§7) |

핵심:

- `.mcp.json` 등록만으론 push 권한 없음 → **`--channels` 로 명시한 서버만** 활성
- room/chat ID 가 아니라 **sender ID** 로 게이트 (그룹 채팅에서 다른 사람 침입 방지)
- Permission Relay 가 켜진 채널은 allowlist 사용자가 **승인/거절 권한도 가짐** → 신뢰하는 사람만 추가

---

## 7. 내 채널 만들기 (Webhook Receiver)

[Channels Reference](https://code.claude.com/docs/en/channels-reference) 의 1-way webhook 예제. CI 실패, 모니터링 알람 등 **HTTP POST 가능한 모든 시스템**을 Claude 에게 push.

### 7-1. 프로젝트 셋업

```bash
mkdir webhook-channel && cd webhook-channel
bun add @modelcontextprotocol/sdk
```

### 7-2. 채널 서버 작성 — `webhook.ts`

```ts
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    // 이 키가 "이건 channel 이다" 라고 알려줌
    capabilities: { experimental: { 'claude/channel': {} } },
    // Claude 시스템 프롬프트에 들어감
    instructions:
      'Events from the webhook channel arrive as <channel source="webhook" ...>. ' +
      'They are one-way: read them and act, no reply expected.',
  },
)

await mcp.connect(new StdioServerTransport())

Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',  // localhost 만
  async fetch(req) {
    const body = await req.text()
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: {
        content: body,
        meta: { path: new URL(req.url).pathname, method: req.method },
      },
    })
    return new Response('ok')
  },
})
```

핵심 포인트:

- `capabilities.experimental['claude/channel'] = {}` 가 채널 등록 트리거
- `instructions` 에 이벤트 형식 / 답할지 말지 / 답한다면 어떤 도구를 어떤 attribute 로 라우팅할지 명시
- `mcp.notification()` 의 `meta` 키는 식별자 문자만 (영문/숫자/`_`). 하이픈 들어가면 silently drop

### 7-3. `.mcp.json` 등록

```json
{
  "mcpServers": {
    "webhook": { "command": "bun", "args": ["./webhook.ts"] }
  }
}
```

### 7-4. 개발용 플래그로 실행

Research Preview 동안 자작 채널은 allowlist 에 없으므로 dev 플래그:

```bash
# .mcp.json 의 bare 서버
claude --dangerously-load-development-channels server:webhook

# 자체 마켓플레이스에 패키징한 플러그인
claude --dangerously-load-development-channels plugin:yourplugin@yourmarketplace
```

> `channelsEnabled` 조직 정책은 여전히 적용됨. 이 플래그는 allowlist 만 우회.

### 7-5. 테스트

```bash
curl -X POST localhost:8788 -d "build failed on main: https://ci.example.com/run/1234"
```

세션에 도착:

```
<channel source="webhook" path="/" method="POST">
build failed on main: https://ci.example.com/run/1234
</channel>
```

### 7-6. 디버깅

| 증상 | 원인 / 확인 |
|---|---|
| `curl` 은 성공인데 Claude 한테 안 옴 | 세션에서 `/mcp` → 서버 status. "Failed to connect" 면 import/의존성 에러. `~/.claude/debug/<session-id>.txt` 에 stderr |
| `curl: connection refused` | 포트가 아직 안 떴거나, 이전 실행의 stale process 가 잡고 있음. `lsof -i :8788` → `kill <pid>` |

---

## 8. 양방향 채널로 확장하기 (Reply Tool)

webhook 예제에 `reply` 도구를 추가해서 Claude 가 답신할 수 있게:

```ts
// Server constructor 에 tools: {} 추가
capabilities: {
  experimental: { 'claude/channel': {} },
  tools: {},
},

// 도구 등록
import { ListToolsRequestSchema, CallToolRequestSchema }
  from '@modelcontextprotocol/sdk/types.js'

mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Send a message back over this channel',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string' },
        text:    { type: 'string' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } =
      req.params.arguments as { chat_id: string; text: string }
    // 실제 chat 플랫폼 API POST 또는 자체 outbound 채널로
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})
```

`instructions` 도 업데이트:

```ts
instructions:
  'Messages arrive as <channel source="webhook" chat_id="...">. ' +
  'Reply with the reply tool, passing the chat_id from the tag.',
```

---

## 9. Permission Relay 직접 추가하기

자작 채널에서 폰/원격 승인 받기:

### 9-1. capability 선언

```ts
capabilities: {
  experimental: {
    'claude/channel': {},
    'claude/channel/permission': {},  // 권한 relay 옵트인
  },
  tools: {},
},
```

> **반드시 sender allowlist 가 있는 채널에서만 켤 것.** 아무나 접근하는 채널에서 켜면 누구나 도구 실행 승인 가능.

### 9-2. 권한 요청 핸들러

```ts
import { z } from 'zod'

const PermissionRequestSchema = z.object({
  method: z.literal('notifications/claude/channel/permission_request'),
  params: z.object({
    request_id:    z.string(),  // 5글자 소문자, 'l' 없음
    tool_name:     z.string(),  // 'Bash' / 'Write' / ...
    description:   z.string(),  // 사람이 읽을 요약
    input_preview: z.string(),  // 인자 JSON, ~200자 truncate
  }),
})

mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
  send(
    `Claude wants to run ${params.tool_name}: ${params.description}\n\n` +
    `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
  )
})
```

### 9-3. inbound 에서 verdict 가로채기

```ts
// "y abcde" / "yes abcde" / "n abcde" / "no abcde"
// [a-km-z]: Claude Code 가 쓰는 ID 알파벳 (소문자, 'l' 제외)
const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i

async function onInbound(message) {
  if (!allowed.has(message.from.id)) return  // sender 게이트 먼저

  const m = PERMISSION_REPLY_RE.exec(message.text)
  if (m) {
    await mcp.notification({
      method: 'notifications/claude/channel/permission',
      params: {
        request_id: m[2].toLowerCase(),
        behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
      },
    })
    return  // verdict 처리, chat 으로는 안 보냄
  }

  // 아니면 일반 chat 으로 forward
  await mcp.notification({
    method: 'notifications/claude/channel',
    params: { content: message.text, meta: { chat_id: String(message.chat.id) } },
  })
}
```

동작:

- 로컬 터미널 다이얼로그도 동시에 열려 있음 → 먼저 답한 쪽 적용, 다른 쪽은 자동 닫힘
- 형식 안 맞는 답 (예: "approve it") → 정규식 fail → 일반 chat 으로 흘러서 Claude 에게 전달
- 형식 맞지만 ID 가 모르는 값 → Claude Code 가 silently drop, 다이얼로그는 그대로

---

## 10. Team / Enterprise 관리자 설정

조직 플랜에선 **기본적으로 채널이 꺼져 있음**. managed settings 두 개로 통제 (사용자 override 불가):

| 설정 | 의미 | 미설정 시 |
|---|---|---|
| `channelsEnabled` | 마스터 스위치. `true` 여야 어떤 채널이든 메시지 도달. dev 플래그까지 차단 가능 | 채널 차단 |
| `allowedChannelPlugins` | 등록 허용 플러그인 목록. 설정하면 Anthropic 기본 allowlist 를 **대체** | Anthropic 기본 목록 적용 |

### 활성화

[claude.ai → Admin settings → Claude Code → Channels](https://claude.ai/admin-settings/claude-code) 토글 또는 managed settings:

```json
{
  "channelsEnabled": true,
  "allowedChannelPlugins": [
    { "marketplace": "claude-plugins-official", "plugin": "telegram" },
    { "marketplace": "claude-plugins-official", "plugin": "discord" },
    { "marketplace": "acme-corp-plugins", "plugin": "internal-alerts" }
  ]
}
```

- `allowedChannelPlugins` 빈 배열 → allowlist 차단 + dev 플래그는 여전히 우회 가능
- 채널 자체를 완전 차단하려면 `channelsEnabled` 를 미설정/`false`

> Pro / Max 개인 사용자는 위 체크 없이 바로 사용 가능.

---

## 11. 슬래시 커맨드 한눈에

| 커맨드 | 채널 | 설명 |
|---|---|---|
| `/plugin install <name>@claude-plugins-official` | 공통 | 채널 플러그인 설치 |
| `/reload-plugins` | 공통 | 설치 후 configure 커맨드 활성화 |
| `/telegram:configure <token>` | Telegram | 봇 토큰 저장 |
| `/telegram:access pair <code>` | Telegram | 페어링 |
| `/telegram:access policy allowlist` | Telegram | 본인만 허용 |
| `/discord:configure <token>` | Discord | 봇 토큰 저장 |
| `/discord:access pair <code>` | Discord | 페어링 |
| `/discord:access policy allowlist` | Discord | 본인만 허용 |
| `/imessage:access allow <handle>` | iMessage | 다른 연락처 허용 |
| `/mcp` | 공통 | 채널 서버 status 확인 |

CLI 플래그:

```bash
claude --channels plugin:<name>@<marketplace> [...]            # 정식 사용
claude --channels plugin:a@m plugin:b@m                        # 다중
claude --dangerously-load-development-channels server:<name>   # 개발 / .mcp.json 직접
claude --dangerously-load-development-channels plugin:<n>@<m>  # 자체 마켓플레이스 dev
claude --dangerously-skip-permissions                          # unattended (위험)
```

---

## 12. 문제 해결

| 증상 | 해결 |
|---|---|
| "blocked by org policy" | 관리자가 `channelsEnabled: true` 켜야 함 |
| "plugin … not on the organization's approved list" | 관리자에게 `allowedChannelPlugins` 추가 요청 또는 승인된 플러그인 사용 |
| 봇이 페어링 코드 안 보냄 | `claude --channels …` 가 켜져 있는지 확인. 채널이 떠 있어야 봇이 응답 |
| Claude 에게 push 가 안 옴 | `/mcp` 로 서버 status. `~/.claude/debug/<session-id>.txt` 에서 stderr 확인 |
| `connection refused` | 이전 실행이 포트 잡고 있음. `lsof -i :<port>` → `kill <pid>` |
| iMessage 가 즉시 종료 (`authorization denied`) | Full Disk Access 미허가. System Settings 에서 터미널 추가 |
| Reply 텍스트가 터미널에 안 보임 | 정상 동작. 답은 외부 플랫폼(Telegram/Discord/...)에서만 표시 |
| 토큰 / Key 인증으로 시도 | 채널은 **claude.ai 로그인 전용**. Console / API Key 미지원 |

---

## 13. 채널을 언제 쓰는가

```
세션 자체를 다른 데서 시작/조종         → Claude Code on the Web / Slack / Remote Control
필요할 때 Claude 가 외부 데이터 조회    → 표준 MCP 서버
타이머로 주기 실행                      → Scheduled Tasks
"이미 열려있는 내 세션"으로 이벤트 push → Channels  ★
```

대표 시나리오:

- **Chat bridge**: 폰에서 Telegram 으로 "지금 디버깅 중인 그 함수 어디까지 했어?" → 로컬 작업 디렉토리 그대로에서 답변
- **Webhook receiver**: CI 실패 알림 → Claude 가 마침 그 PR 보고 있던 세션에서 즉시 reproduce / fix 시도
- **Monitoring**: Sentry / Grafana 알람 → 서비스 코드 컨텍스트가 살아있는 세션에 push → 원인 분석 시작

---

## 14. 다음 단계

- [공식 채널 플러그인 소스](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins) — Telegram / Discord / iMessage / fakechat 전체 구현 (페어링 / 첨부 / 메시지 편집 포함)
- [Channels Reference](https://code.claude.com/docs/en/channels-reference) — 자작 채널 풀 레퍼런스
- 채널을 [Plugin](https://code.claude.com/docs/en/plugins) 으로 패키징 → [Marketplace](https://code.claude.com/docs/en/plugin-marketplaces) 배포 → [공식 마켓플레이스에 제출](https://code.claude.com/docs/en/plugins#submit-your-plugin-to-the-official-marketplace) (보안 리뷰 후 allowlist 등재)
- 이슈 / 피드백: [claude-code GitHub](https://github.com/anthropics/claude-code/issues)

수고하셨습니다. 🎉
