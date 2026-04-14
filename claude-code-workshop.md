# Claude Code 기본 워크샵

Anthropic 계정 생성부터 요금제, macOS / Windows 설치, 인증, 기본 사용법, 파일 변경사항 보기 → 커밋 → 푸시, 추천 설정까지 진행합니다.

> Git 설치 / GitHub 계정 / SSH 키 설정은 [[github-workshop]] 를 먼저 완료해 주세요.

---

## 1. Claude Code 란?

- Anthropic 에서 만든 **터미널 기반 AI 코딩 에이전트**
- 내 프로젝트 폴더를 직접 읽고/수정/실행할 수 있음 (파일, Git, 셸 명령)
- VSCode / JetBrains 확장, 웹(claude.ai/code) 으로도 사용 가능
- 모델: **Claude Opus 4.6**, **Sonnet 4.6**, **Haiku 4.5**

---

## 2. Anthropic 계정 만들기 & 요금제

### 2-1. 계정 생성

1. [https://claude.ai](https://claude.ai) 접속 → `Sign up`
2. 이메일 / 구글 계정으로 가입 → 이메일 인증

### 2-2. 요금제 (2026년 기준, 변동 가능)

| 플랜 | 대상 | 비고 |
|---|---|---|
| **Free** | 체험용 | Claude Code 사용 제한적 |
| **Pro** ($20/월) | 개인 | Claude Code 사용 가능, 일반적인 개발 용도 |
| **Max** ($100~200/월) | 파워유저 | 사용량 크게 확대 |
| **Team / Enterprise** | 팀/회사 | 관리자 기능, SSO |
| **API (종량제)** | 스크립트/자동화 | Console 에서 API Key 발급, 사용량만큼 과금 |

> Claude Code 는 **Pro 이상 구독** 또는 **API Key** 둘 중 하나로 인증하면 사용 가능.

최신 요금은 [https://www.anthropic.com/pricing](https://www.anthropic.com/pricing) 에서 확인.

---

## 3. 설치

### 3-1. 사전 요구사항

- **Node.js 18+** 필요
  ```bash
  node --version
  ```
  없다면:
  - macOS: `brew install node`
  - Windows: [https://nodejs.org](https://nodejs.org) LTS 설치

### 3-2. macOS

```bash
# npm 전역 설치
npm install -g @anthropic-ai/claude-code

# 설치 확인
claude --version
```

### 3-3. Windows

`PowerShell` 또는 `Git Bash` 에서:

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

> 설치 중 권한 에러가 나면 PowerShell 을 **관리자 권한**으로 실행.

---

## 4. 최초 실행 & 로그인

프로젝트 폴더로 이동해서 실행합니다.

```bash
cd ~/Desktop/<your-project>
claude
```

첫 실행 시:

1. 로그인 방식 선택
   - `Log in with Anthropic account` (Pro/Max/Team 구독자)
   - `Use API key` (Console 에서 발급한 키 입력)
2. 브라우저가 열리고 인증 후 자동으로 돌아옴

로그인되면 터미널에 프롬프트가 뜹니다:

```
> _
```

---

## 5. 기본 사용법

자연어로 지시하면 됩니다.

```
> README.md 요약해줘
> src/utils.ts 의 formatDate 함수 버그 고쳐줘
> 새 API 엔드포인트 /users/me 를 추가하고 테스트도 같이 짜줘
> 이 프로젝트 전체 구조 설명해줘
```

Claude 가 필요하면 **파일 읽기 / 편집 / 셸 명령 실행**을 알아서 수행하며, 위험한 작업 전에는 승인을 요청합니다.

### 자주 쓰는 슬래시 커맨드

| 명령 | 설명 |
|---|---|
| `/help` | 도움말 |
| `/clear` | 대화 히스토리 초기화 |
| `/compact` | 긴 대화 압축 |
| `/model` | 사용할 모델 변경 (Opus/Sonnet/Haiku) |
| `/config` | 설정 열기 |
| `/cost` | 현재 세션 토큰/비용 보기 |
| `/login`, `/logout` | 계정 전환 |
| `/exit` | 종료 |

### 유용한 키

- `!<명령>` : 해당 줄을 셸 명령으로 실행 (예: `!git status`)
- `Shift + Tab` : 권한 모드 전환 (승인 필요 / 자동 실행)
- `Esc` : 현재 작업 중단
- `↑` / `↓` : 이전 프롬프트 히스토리

---

## 6. 파일 변경사항 다루기 (Claude Code 방식)

Claude 가 파일을 수정하면:

1. **diff 형태로 변경 제안**을 보여줌
2. 사용자가 `Accept` / `Reject` 선택
3. 승인한 것만 실제 파일에 반영

즉, `git add` 이전 단계에서 이미 "한 번 더 리뷰"가 들어갑니다.

### 6-1. 변경사항 확인

```
> 방금 바꾼 내용 요약해줘
> 지금까지 수정한 파일 목록 보여줘
```

또는 터미널에서 직접:

```bash
!git status
!git diff
```

### 6-2. 스테이지 → 커밋 → 푸시

Claude 에게 바로 시킬 수 있습니다.

```
> 지금까지 변경된 내용들 의미 단위로 나눠서 커밋 메시지 제안해줘
> 좋아, 그대로 커밋해줘
> main 으로 푸시해줘
```

또는 수동으로:

```bash
!git add .
!git commit -m "feat: ..."
!git push origin main
```

> 보통은 "Claude 가 제안 → 내가 승인 → Claude 가 실행" 흐름을 씁니다.

---

## 7. 권한 모드 이해하기

Claude Code 는 위험한 작업을 **임의로 실행하지 않습니다**. 모드는 `Shift + Tab` 으로 전환.

| 모드 | 설명 |
|---|---|
| **Default** | 파일 읽기 등 안전한 작업은 자동, 쓰기/실행은 묻기 |
| **Auto-accept edits** | 파일 편집까지 자동 승인 (실습/샌드박스에서 편함) |
| **Plan mode** | 실행 없이 "계획"만 세움. 승인하면 실행 |

> 중요한 리포에서는 **Default** 유지 권장. 데모용 샌드박스에서만 Auto-accept.

---

## 8. 추천 설정 & 파일

### 8-1. `CLAUDE.md` (프로젝트 메모리)

프로젝트 루트에 `CLAUDE.md` 를 두면 Claude 가 매 세션 자동으로 읽습니다.

```markdown
# 프로젝트 규칙

- 패키지 매니저는 pnpm 사용
- 테스트는 vitest, 파일명은 *.test.ts
- 커밋 메시지는 conventional commits
- 외부 API 호출 전 반드시 캐싱 레이어 확인
```

### 8-2. `.claude/settings.json`

권한 / 환경변수 / 훅을 설정.

```json
{
  "permissions": {
    "allow": ["Bash(git *)", "Bash(pnpm *)"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

### 8-3. IDE 연동

- VSCode: 확장 마켓에서 `Claude Code` 설치 → 에디터 안에서 바로 호출
- JetBrains: Plugin Marketplace 에서 `Claude Code`

---

## 9. 실습: hello.md 만들고 커밋까지

1. 빈 폴더 만들기
   ```bash
   mkdir ~/Desktop/claude-demo && cd ~/Desktop/claude-demo
   git init
   claude
   ```
2. Claude 에게:
   ```
   > hello.md 라는 파일을 만들고 Claude Code 워크샵 첫날이라는 소개 글을 영어/한국어로 써줘
   ```
3. 제안된 diff 를 `Accept`
4. 이어서:
   ```
   > 지금 변경사항 git status 로 확인하고, 적절한 메시지로 커밋해줘
   ```
5. (선택) GitHub 저장소를 만들고:
   ```
   > 이 로컬 저장소를 git@github.com:<me>/claude-demo.git 에 연결해서 푸시해줘
   ```

---

## 10. 자주 쓰는 패턴

```
> 이 에러 로그 원인 찾아줘: <붙여넣기>
> tests/ 폴더 전체 실행하고 실패하는 것만 고쳐줘
> 이 PR 에 리뷰 코멘트 달아줘 (gh pr view 123)
> 방금 만든 함수에 타입 붙이고 jsdoc 도 추가해줘
> 이 파일 리팩터링하되 동작은 그대로 유지되게 해줘
```

---

## 11. 문제 해결

- **`claude: command not found`**
  → `npm install -g @anthropic-ai/claude-code` 재실행, Node 버전 확인
- **로그인이 안 됨 / 브라우저가 안 열림**
  → `/logout` 후 `/login` 재시도. API Key 방식으로 우회 가능
- **파일 수정이 자꾸 거부됨**
  → 권한 모드 확인(`Shift + Tab`), `.claude/settings.json` 의 `deny` 규칙 확인
- **토큰을 너무 빨리 소진**
  → `/compact` 로 대화 압축, 큰 파일은 일부만 보여주기, 모델을 Sonnet/Haiku 로 변경

---

## 12. 전체 흐름 요약

```
Anthropic 가입 → 요금제 선택(Pro 이상 or API Key)
     ↓
Node 설치 → npm i -g @anthropic-ai/claude-code
     ↓
프로젝트 폴더에서 claude 실행 → 로그인
     ↓
자연어로 지시 → diff 리뷰 → Accept
     ↓
!git add / commit / push 또는 Claude 에게 시키기
     ↓
CLAUDE.md 로 프로젝트 규칙 쌓기 🎉
```

수고하셨습니다! 🎉
