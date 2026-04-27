# MCP Setup (Model Context Protocol)

MCP는 LLM을 외부 시스템(파일, API, DB, SaaS)에 **표준화된 방식으로 연결**하는 프로토콜. Anthropic이 제안했고 Claude Code, Codex, Gemini CLI, OpenCode, Crush 등 대부분의 모던 에이전트가 지원합니다.

---

## 왜 쓰나

기본 에이전트는 파일 시스템과 셸만 다룰 수 있음. MCP를 붙이면:

- **Notion / Confluence / Jira** — 문서 조회 / 티켓 처리
- **Slack / Discord / Gmail** — 메시지 검색 / 발송
- **Figma** — 디자인 컨텍스트 / 컴포넌트 매핑
- **GitHub** — Issue / PR / 액션
- **DB (Postgres/MySQL/BigQuery)** — 직접 쿼리
- **사내 API** — 직접 연결

이게 다 **에이전트의 도구**로 노출됩니다.

---

## 동작 모델

```
Claude Code (host) ←→ MCP Server ←→ 외부 시스템
                       (별도 프로세스)
```

서버는 stdio 또는 HTTP로 host와 통신. host는 서버가 광고하는 tools / resources / prompts를 자동으로 도구로 등록.

---

## 서버 종류

### 1. 공식 서버 (`@modelcontextprotocol/server-*`)

```bash
# 파일 시스템
npx -y @modelcontextprotocol/server-filesystem ~/Documents

# Git
npx -y @modelcontextprotocol/server-git --repository .

# Postgres
npx -y @modelcontextprotocol/server-postgres "postgresql://..."

# 슬랙
npx -y @modelcontextprotocol/server-slack

# 브라우저 자동화 (Puppeteer)
npx -y @modelcontextprotocol/server-puppeteer
```

### 2. 벤더 공식

| 서비스 | 비고 |
|---|---|
| Notion | https://mcp.notion.com (HTTP) |
| Atlassian (Jira/Confluence) | 클라우드 OAuth |
| Figma | 데스크톱 앱과 연동, 디자인 컨텍스트 |
| Linear | https://mcp.linear.app |
| GitHub | OAuth 또는 PAT |

### 3. 커뮤니티

`awesome-mcp-servers` 류 큐레이트 리스트 다수 — 실제 쓰는 것만 골라서.

---

## 설치 (Claude Code 기준)

### CLI로

```bash
# stdio 서버 (로컬 프로세스)
claude mcp add filesystem -- \
  npx -y @modelcontextprotocol/server-filesystem ~/Documents

# HTTP 서버 (원격)
claude mcp add notion --transport http --url https://mcp.notion.com

# 환경변수 전달
claude mcp add postgres --env DATABASE_URL=$DATABASE_URL -- \
  npx -y @modelcontextprotocol/server-postgres
```

### 직접 settings.json 편집

`~/.claude/settings.json` 또는 `<project>/.claude/settings.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/kang/Documents"]
    },
    "notion": {
      "type": "http",
      "url": "https://mcp.notion.com"
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": { "DATABASE_URL": "$DATABASE_URL" }
    }
  }
}
```

### 확인

```bash
claude mcp list
```

또는 세션 안에서 `/mcp` — 상태 / 사용 가능한 도구 확인.

---

## 다른 에이전트에서

### Gemini CLI

`~/.gemini/settings.json` 의 `mcpServers` 키. 형식 거의 동일.

### OpenCode

`config.json` 의 `mcp` 배열.

```json
{
  "mcp": [
    { "name": "filesystem", "command": ["npx","-y","@modelcontextprotocol/server-filesystem","~/Documents"] }
  ]
}
```

### Codex

`~/.codex/config.toml`:

```toml
[mcp.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/Users/kang/Documents"]
```

### Crush

`~/.config/crush/config.json` 의 `mcp` 항목.

---

## 권한 / 보안

MCP 서버는 보통 **에이전트가 호출할 때마다 사용자 승인**을 받게 설정합니다.

```json
{
  "permissions": {
    "allow": [
      "mcp__notion__search",
      "mcp__filesystem__read_file"
    ],
    "deny": [
      "mcp__postgres__execute"
    ]
  }
}
```

**주의**:
- 사내 데이터에 액세스 가능한 MCP는 키/토큰을 환경변수로만 주입 (settings.json에 평문 금지)
- 외부 호스팅 MCP 서버는 **데이터가 어디로 가는지** 확인 (LLM 회사가 아닌 제3자에 흐를 수 있음)
- Production DB MCP는 read-only 권한으로

---

## 자주 쓰는 조합 (실전 예)

### 풀스택 개발자

```
filesystem (코드)
git (히스토리)
postgres (스키마/데이터 조회)
github (PR/Issue)
```

### 디자인 → 코드 워크플로우

```
figma (디자인 컨텍스트)
filesystem (코드)
git
```

### PM / 문서 작업

```
notion
slack
gmail
github
```

### 데이터 분석

```
postgres / bigquery
filesystem
puppeteer (대시보드 캡처)
```

---

## 디버깅

서버가 인식 안 될 때:

```bash
# 직접 띄워보기
npx -y @modelcontextprotocol/server-filesystem ~/Documents

# 로그
claude --debug
# 또는 MCP 서버 자체 로그를 stderr로 받아 확인
```

설정 변경 후 에이전트 재시작 필요한 경우 많음 (`/mcp restart` 가능한 에이전트도 있음).

---

## 직접 MCP 서버 만들기

Python / TypeScript SDK 제공.

```bash
npm create @modelcontextprotocol/server my-server
```

가장 간단한 서버는 50줄 미만. 사내 API 같은 거 붙이기에 매우 좋음.

---

## 참고

- 공식 사양: https://modelcontextprotocol.io
- Awesome MCP servers: 검색하면 여러 큐레이트 리스트 나옴
- Hooks와 함께: [`./hooks.md`](./hooks.md) — MCP 호출 시 추가 검증/알림 가능
