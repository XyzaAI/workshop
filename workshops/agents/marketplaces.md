# AI 에이전트 마켓플레이스 비교

Claude Code, Codex, Gemini CLI 모두 2026년 기준 **공식 마켓플레이스**를 제공한다. 스킬·MCP 서버·커맨드·서브에이전트·훅을 패키지로 묶어 공유/설치할 수 있다.

| 도구 | 마켓플레이스 명 / URL | 단위 호칭 | 단일 매니페스트 | 커스텀 마켓 추가 |
|---|---|---|---|---|
| **Claude Code** | `claude-plugins-official` ([claude.com/plugins](https://claude.com/plugins)) | plugin | `.claude-plugin/marketplace.json` | GitHub `owner/repo`, Git URL, 로컬 경로, marketplace.json URL |
| **Codex** | 내장 Codex Plugin Directory ([developers.openai.com/codex/plugins](https://developers.openai.com/codex/plugins)) | plugin | `marketplace.json` | GitHub, Git URL, 로컬 디렉토리, marketplace.json URL |
| **Gemini CLI** | Extensions Gallery ([geminicli.com/extensions/](https://geminicli.com/extensions/)) | extension | `gemini-extension.json` | Git URL 직접 install |

## Claude Code

공식 마켓 `claude-plugins-official`은 시작하면 자동으로 등록된다.

```bash
# 설치
/plugin install github@claude-plugins-official

# 마켓 추가 (커스텀)
/plugin marketplace add anthropics/claude-code           # GitHub
/plugin marketplace add https://gitlab.com/x/plugins.git # Git URL
/plugin marketplace add ./my-marketplace                 # 로컬
/plugin marketplace add https://example.com/marketplace.json

# 관리
/plugin                       # 인터랙티브 UI (Discover/Installed/Marketplaces/Errors)
/plugin marketplace list
/plugin marketplace update <name>
/plugin marketplace remove <name>
/plugin disable | enable | uninstall <name>@<marketplace>
/reload-plugins               # 재시작 없이 적용
```

스코프: `--scope user|project|local` (project는 `.claude/settings.json`에 기록되어 팀 공유)

팀 공통 마켓 자동 설치는 `.claude/settings.json`의 `extraKnownMarketplaces`로 설정.

공식 카테고리: Code intelligence (LSP), External integrations (GitHub/Slack/Linear/Notion 등), Development workflows (`commit-commands`, `pr-review-toolkit`), Output styles.

공식 등록 신청: [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit) 또는 [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit).

## Codex

2026년 3월 26일 정식 출시. App / CLI / IDE 확장 모두에서 작동.

```bash
# 인터랙티브
/plugins                      # 마켓플레이스별 탭, 활성/비활성 토글, 설치/제거

# 호출
@<plugin>                     # 명시적 호출
"Summarize unread Gmail threads"   # 자연어로 호출 가능
```

마켓 소스: 0.122.0부터 `marketplace add`로 remote / cross-repo / 로컬 마켓플레이스 manifest 추가 가능 (GitHub, Git URL, 로컬 디렉토리, marketplace.json URL). 0.124.0에서 hooks 안정화 (config.toml 인라인 설정), 0.125.0에서 원격 플러그인 설치 + 마켓 업그레이드 지원.

비활성화: `~/.codex/config.toml`에 `enabled = false` 또는 플러그인 브라우저에서 uninstall.

공식 디렉토리에 포함된 통합: Box, Figma, Linear, Notion, Sentry, Slack, Gmail, Hugging Face 등.

## Gemini CLI

공식 갤러리: [geminicli.com/extensions/browse/](https://geminicli.com/extensions/browse/). 0.36.0 (2026-04-01)에서 multi-registry 아키텍처 도입.

```bash
# 설치
gemini extensions install https://github.com/gemini-cli-extensions/workspace

# 관리
gemini extensions list
/extensions list              # 인터랙티브
```

확장이 묶는 것: prompts, MCP servers, custom commands, themes, hooks, sub-agents, agent skills.

설치 시 prompt 기능: 확장이 정의한 설정 값을 사용자에게 입력받아 적용.

주의: Gallery는 third-party 확장 다수 포함, Google이 검증/보증하지 않음.

## 우리 팀 적용

- 회사 표준은 **`agent-hub`** 한 군데에 모으고, 각 도구 마켓플레이스 manifest를 그 레포 안에 둠
  - `agent-hub/claude-plugins/.claude-plugin/marketplace.json`
  - `agent-hub/codex-plugins/marketplace.json`
  - `agent-hub/gemini-extensions/` (개별 extension 레포 모음)
- 팀별 plugin/extension은 **`xyza-plugins`** (별도 레포)에 팀 디렉토리로 분리
- 신규 도구가 같은 매니페스트 패턴(JSON 카탈로그 + 단위 manifest)을 채택하면 동일 구조로 흡수

## 참고 링크

- Claude Code: <https://code.claude.com/docs/en/discover-plugins>
- Claude Code marketplace 만들기: <https://code.claude.com/docs/en/plugin-marketplaces>
- Codex plugins: <https://developers.openai.com/codex/plugins>
- Codex changelog: <https://developers.openai.com/codex/changelog>
- Gemini CLI extensions: <https://geminicli.com/docs/extensions/>
- Gemini extensions gallery: <https://geminicli.com/extensions/>
