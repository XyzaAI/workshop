# Claude Code

Anthropic의 공식 CLI AI 코딩 에이전트. 가장 안정적이고 생태계가 풍부합니다.

> 핸즈온 가이드는 [`../../workshops/claude-code-workshop.md`](../../workshops/claude-code-workshop.md) 참고. 이 문서는 **레퍼런스 / 심화** 위주.

---

## 핵심 특징

- **모델**: Claude Opus 4.7, Sonnet 4.6, Haiku 4.5 (1M 컨텍스트 지원 모델 포함)
- **런타임**: Node.js 18+, `npm i -g @anthropic-ai/claude-code`
- **인증**: Pro/Max/Team 구독 또는 Anthropic Console API Key
- **인터페이스**: 터미널, VSCode/JetBrains 확장, [claude.ai/code](https://claude.ai/code) 웹

---

## 디렉토리 구조

```
~/.claude/                 # 사용자 글로벌 설정
├── settings.json          # 권한, 환경변수, 훅
├── CLAUDE.md              # 모든 세션에 주입되는 메모리
├── commands/              # 사용자 슬래시 커맨드 (.md 파일)
├── agents/                # 사용자 서브에이전트 정의
└── skills/                # 사용자 스킬

<project>/.claude/         # 프로젝트별 설정 (커밋해서 팀과 공유)
├── settings.json
├── settings.local.json    # 개인용, .gitignore 추천
├── commands/
├── agents/
└── skills/

<project>/CLAUDE.md        # 프로젝트별 메모리
```

---

## 4대 확장 포인트

Claude Code를 "내 워크플로우"에 맞추려면 이 4개를 알아야 합니다.

### 1. CLAUDE.md (메모리)

세션 시작 시 자동 주입되는 프롬프트. 프로젝트 규칙, 팀 컨벤션, 자주 헷갈리는 도메인 지식을 적어둡니다.

```markdown
# 프로젝트 규칙
- 패키지 매니저: pnpm
- 테스트: vitest, *.test.ts
- 커밋: conventional commits
- DB 마이그레이션 전 반드시 staging 검증
```

### 2. Slash Commands (`commands/*.md`)

자주 쓰는 프롬프트를 `/이름` 으로 호출. `.md` 파일 한 개 = 슬래시 커맨드 한 개.

```markdown
---
description: PR 리뷰
---

현재 브랜치의 변경사항을 리뷰해줘. 다음을 체크:
- 보안 이슈 (XSS/SQLi/인젝션)
- 테스트 커버리지
- 네이밍/타입 일관성
```

### 3. Hooks (`settings.json`)

특정 이벤트(파일 수정, 명령 실행 등)에 셸 명령 자동 실행.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "pnpm lint --fix" }
        ]
      }
    ]
  }
}
```

자세히는 [`../tips/hooks.md`](../tips/hooks.md).

### 4. MCP (Model Context Protocol)

외부 시스템(Slack/Notion/Figma/DB 등)을 도구로 연결.

```bash
claude mcp add notion --transport http --url https://mcp.notion.com
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Documents
```

자세히는 [`../tips/mcp-setup.md`](../tips/mcp-setup.md).

---

## 권한 모드 (Shift+Tab으로 전환)

| 모드 | 동작 | 추천 상황 |
|---|---|---|
| **Default** | 안전 작업 자동, 위험 작업은 묻기 | 운영/프로덕션 코드 |
| **Auto-accept edits** | 파일 편집까지 자동 | 실험/샌드박스 |
| **Plan mode** | 실행 없이 계획만 | 큰 변경 전 사전 검토 |

`settings.json`의 `permissions.allow` / `deny`로 더 세밀하게 제어 가능.

```json
{
  "permissions": {
    "allow": ["Bash(git *)", "Bash(pnpm *)", "Edit", "Write"],
    "deny": ["Bash(rm -rf *)", "Bash(sudo *)"]
  }
}
```

---

## 자주 쓰는 슬래시 커맨드

| 명령 | 설명 |
|---|---|
| `/help` | 도움말 |
| `/model` | 모델 변경 (Opus/Sonnet/Haiku) |
| `/compact` | 긴 대화 압축 (토큰 절약) |
| `/clear` | 대화 초기화 |
| `/cost` | 현재 세션 토큰/비용 |
| `/config` | 설정 메뉴 |
| `/agents` | 서브에이전트 목록/관리 |
| `/mcp` | MCP 서버 상태 |
| `/review` | 변경사항 리뷰 |
| `/init` | 프로젝트에 CLAUDE.md 생성 |

---

## 서브에이전트

특정 작업에 특화된 보조 에이전트를 정의해서 메인 컨텍스트를 보호합니다.

```markdown
---
name: code-reviewer
description: 큰 변경 후 코드 리뷰 전담
tools: Read, Grep, Bash
---

당신은 시니어 리뷰어입니다. 다음을 검사하세요:
- 보안 취약점
- 성능 회귀
- 테스트 누락
```

`.claude/agents/code-reviewer.md`로 저장하면 자동 인식.

---

## 자주 쓰는 패턴

```
> 이 에러 로그 원인 찾아줘: <붙여넣기>
> tests/ 전체 돌리고 실패만 고쳐줘
> gh pr view 123 리뷰해줘
> 이 함수에 타입 + jsdoc 붙여줘
> 동작 유지하면서 리팩터링
> 지금 변경사항 의미 단위로 나눠 커밋해줘
```

---

## 토큰/비용 절약 팁

- 큰 파일은 `Read` 시 `offset/limit` 으로 일부만
- 긴 세션은 `/compact` 정기적으로
- 단순 작업은 Sonnet, 추론 무거운 건 Opus
- `permissions.allow` 적극 등록 → 매번 묻는 비용 절감 (시간/토큰 둘 다)
- 파일 검색은 `Grep`/`Glob` 우선, `cat` 같은 셸 도구 지양

---

## IDE 통합

- **VSCode**: 마켓플레이스 `Claude Code` 설치. 에디터에서 선택한 코드를 바로 컨텍스트로.
- **JetBrains**: 플러그인 마켓플레이스에서 `Claude Code`.
- **iTerm/터미널**: 그냥 `claude` 실행하면 됨.

---

## 참고

- 공식 문서: https://docs.claude.com/en/docs/claude-code
- GitHub Issues: https://github.com/anthropics/claude-code/issues
- 활용 팁: [`../tips/`](../tips/) 폴더 전체
