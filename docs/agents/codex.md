# Codex CLI

OpenAI의 공식 코딩 에이전트. ChatGPT 계정과 직접 연결되어 GPT-5/o-시리즈 모델을 터미널/IDE에서 사용합니다.

---

## 핵심 특징

- **모델**: GPT-5, o3-mini, o4-mini 등 OpenAI 최신 모델
- **인증**: ChatGPT Plus/Pro/Team/Enterprise 또는 OpenAI API Key
- **인터페이스**: CLI, VSCode 확장, ChatGPT 웹 (Codex 탭)
- **런타임**: Node.js 18+

---

## 설치

```bash
npm install -g @openai/codex

codex --version
```

---

## 인증

### ChatGPT 계정 (추천)

```bash
codex login
# 브라우저에서 OAuth 진행
```

ChatGPT Plus 이상이면 별도 과금 없이 일정 한도 사용 가능.

### API Key

```bash
export OPENAI_API_KEY="sk-..."
```

---

## 기본 사용

```bash
cd ~/your-project
codex
```

대화형 TUI가 뜨고 자연어로 지시하면 됩니다. 파일 편집/명령 실행은 모드별로 승인 정책이 다릅니다.

---

## 실행 모드

| 모드 | 설명 |
|---|---|
| **Suggest** (기본) | 파일 변경/명령 실행 전 매번 묻기 |
| **Auto-edit** | 편집은 자동, 명령은 묻기 |
| **Full-auto** | 격리된 sandbox 안에서 자동 실행 |

`--auto` / `--full-auto` 플래그 또는 TUI 안에서 전환.

---

## 설정

```
~/.codex/
├── config.toml      # 글로벌 설정
└── instructions.md  # 글로벌 메모리

<project>/.codex/
└── config.toml
```

`config.toml` 예시:

```toml
model = "gpt-5"
approval_policy = "auto-edit"

[sandbox]
enabled = true
network = "off"
```

---

## Sandbox

Full-auto 모드에서는 OS 레벨 격리(macOS Seatbelt, Linux Landlock)로 안전하게 실행합니다.

- 파일 시스템: 작업 디렉토리만 쓰기 가능
- 네트워크: 기본 차단 (옵션으로 허용)

대규모 자동 리팩터링이나 테스트 실행에 적합.

---

## Claude Code와 비교

| 항목 | Codex | Claude Code |
|---|---|---|
| 베이스 모델 | GPT-5, o-시리즈 | Claude 4.x |
| 추론 무거운 작업 | o3/o4가 강점 | Opus가 강점 |
| 도구 생태계 | MCP 지원 | MCP/Skills/Hooks 풍부 |
| 격리 실행 | Sandbox 강력 | Hooks + 권한 시스템 |
| ChatGPT 연동 | 네이티브 | 없음 |

**선택 가이드**: 이미 ChatGPT Plus를 쓰고 있다면 Codex가 추가 비용 없이 시작하기 좋음. 코딩 워크플로우 정밀도/생태계는 Claude Code 우위.

---

## 자주 쓰는 명령

```
> 이 버그 재현하고 고쳐줘
> tests 디렉토리 전부 통과시켜줘
> git status 보고 의미 단위로 커밋해줘
> 이 함수의 시간복잡도 분석해줘
```

---

## 참고

- GitHub: https://github.com/openai/codex
- 공식 문서: https://platform.openai.com/docs/codex
