# Gemini CLI

Google에서 공식 제공하는 Gemini 기반 터미널 코딩 에이전트. **Google AI Studio API Key가 무료**라서 진입장벽이 낮습니다.

---

## 핵심 특징

- **모델**: Gemini 2.5 Pro, Flash, Flash-Lite
- **컨텍스트 윈도우**: 1M~2M 토큰 (모델 중 가장 큼)
- **인증**:
  - Google 계정 OAuth (개인 사용, 일정 한도 무료)
  - Google AI Studio API Key (무료 티어 존재)
  - Vertex AI / Gemini for Workspace (조직)
- **런타임**: Node.js 18+

---

## 설치

```bash
# npm 전역
npm install -g @google/gemini-cli

# 또는 Homebrew
brew install gemini-cli

# 확인
gemini --version
```

---

## 인증

### 방법 1: Google 계정 (가장 쉬움)

```bash
gemini
# 첫 실행 시 브라우저 OAuth 진행
```

### 방법 2: API Key

[Google AI Studio](https://aistudio.google.com/apikey) 에서 키 발급 후:

```bash
export GEMINI_API_KEY="..."
# 또는 ~/.gemini/.env 에 저장
```

### 방법 3: Vertex AI

```bash
export GOOGLE_GENAI_USE_VERTEXAI=true
export GOOGLE_CLOUD_PROJECT="your-project"
export GOOGLE_CLOUD_LOCATION="us-central1"
gcloud auth application-default login
```

---

## 기본 사용

```bash
cd ~/your-project
gemini
```

```
> README.md 요약
> src/ 전체 구조 보여줘
> 이 함수 테스트 짜줘
```

---

## Claude Code와 비교

| 항목 | Claude Code | Gemini CLI |
|---|---|---|
| 무료 시작 | API Key 필요 (유료) | Google 계정 OAuth로 가능 |
| 컨텍스트 | 200K~1M | 1M~2M |
| 코딩 품질 | 매우 강함 | 강함 (수학/추론에서 특히 좋음) |
| 생태계 | MCP, Skills, Hooks 풍부 | MCP 지원, 툴 직접 정의 가능 |
| 권한/안전 | 정교함 (Shift+Tab 모드) | 비교적 간단 |

**실무 팁**: 둘 다 깔아두고 작업 성격에 따라 골라 씀. 큰 코드베이스를 통째로 읽혀야 할 땐 Gemini, 정밀한 편집/검토는 Claude Code.

---

## 설정 파일

```
~/.gemini/
├── settings.json       # 글로벌 설정
├── .env                # 환경변수
└── GEMINI.md           # 글로벌 메모리 (Claude의 CLAUDE.md 대응)

<project>/.gemini/
├── settings.json
└── GEMINI.md           # 프로젝트 메모리
```

`GEMINI.md` 예시:

```markdown
# 프로젝트 규칙
- TypeScript strict mode
- 테스트 프레임워크: jest
- 커밋 메시지: 한국어로
```

---

## MCP 연결

Claude Code와 거의 동일한 MCP 표준을 따릅니다.

```bash
# settings.json 에 직접 추가하거나
gemini mcp add filesystem npx -y @modelcontextprotocol/server-filesystem ~/Documents
```

---

## 쓸 만한 차별점

- **Google Search 내장 도구** — 최신 정보 조회를 모델이 직접 수행
- **이미지/PDF/오디오 멀티모달** — 스크린샷 붙여넣고 "이 UI 구현해줘" 가능
- **Sandbox 모드** — `--sandbox` 로 격리된 환경에서 실행

---

## 슬래시 커맨드

```
/help
/clear
/compress       # 대화 압축
/memory show    # GEMINI.md 확인
/memory add     # 메모리 추가
/tools          # 사용 가능한 도구 목록
/auth           # 계정/키 변경
/quit
```

---

## 참고

- 공식: https://github.com/google-gemini/gemini-cli
- API Key 발급: https://aistudio.google.com/apikey
- 가격: https://ai.google.dev/pricing
