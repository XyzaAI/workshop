# Aider

가장 오래되고 성숙한 오픈소스 CLI 코딩 에이전트. Python 기반, **Git 친화적**이며 모델 백엔드를 자유롭게 고를 수 있습니다.

---

## 핵심 특징

- **언어**: Python (`pip install aider-chat`)
- **모델**: Claude / GPT / Gemini / Grok / 로컬(Ollama, LM Studio) 모두 지원
- **Git 통합**: 모든 변경을 자동으로 커밋. 매 요청 = 한 커밋.
- **Repo Map**: 큰 프로젝트도 토큰 효율적으로 처리하는 자체 인덱스

---

## 설치

```bash
# pipx 추천 (격리된 환경)
brew install pipx
pipx install aider-chat

# 또는 pip
python -m pip install aider-chat

aider --version
```

---

## 첫 실행

```bash
cd ~/your-project
export ANTHROPIC_API_KEY="..."   # 또는 OPENAI_API_KEY 등
aider
```

기본은 환경변수에서 키를 자동 감지해서 적절한 모델을 선택합니다.

명시적으로:

```bash
aider --model sonnet                       # Claude Sonnet
aider --model gpt-5
aider --model gemini/gemini-2.5-pro
aider --model openai/grok-code-fast-1 \
      --openai-api-base https://api.x.ai/v1
```

---

## Git 워크플로우

**핵심 차별점**: aider는 매 요청마다 자동으로 git commit을 만듭니다.

```
> add a /health endpoint
# aider 가 코드 수정 → 자동 커밋 "feat: add /health endpoint"

> revert last
# 직전 커밋 되돌림
```

마음에 안 들면 `git reset HEAD~` 하면 됩니다. 매우 안전.

`/undo` 슬래시 커맨드로 직전 변경 즉시 되돌리기도 가능.

---

## 자주 쓰는 슬래시 커맨드

```
/add <file>      # 파일을 컨텍스트에 추가
/drop <file>     # 컨텍스트에서 제거
/ls              # 추가된 파일 목록
/diff            # 마지막 변경 diff
/undo            # 마지막 커밋 되돌리기
/commit          # 수동 커밋
/run <cmd>       # 셸 명령 실행
/test <cmd>      # 테스트 실행 + 실패 시 자동 수정
/help
/model           # 모델 변경
/clear
/exit
```

---

## 강력한 기능들

### 1. `/test` 자동 루프

```
/test pytest tests/
```

테스트 실행 → 실패하면 결과를 모델에게 자동 전달 → 수정 → 다시 실행. **TDD 자동화**.

### 2. Repo Map

큰 코드베이스에서도 모든 파일을 컨텍스트에 안 넣고 함수 시그니처/구조만 요약해서 전달. 토큰 효율적.

### 3. Voice 입력

```bash
aider --voice
```

마이크로 말하면 받아쓰기. 긴 지시에 편리.

### 4. Watch 모드

```bash
aider --watch-files
```

에디터에서 `# AI: ...` 주석을 쓰면 aider가 감지해서 그 부분 작업.

---

## 설정

`.aider.conf.yml` (프로젝트 루트):

```yaml
model: sonnet
auto-commits: true
dirty-commits: true
test-cmd: "pnpm test"
lint-cmd: "pnpm lint"
```

`.aider.model.metadata.json` 으로 커스텀 모델 추가 가능.

---

## Claude Code와의 차이

| 항목 | Aider | Claude Code |
|---|---|---|
| 모델 | 어떤 거든 | Claude 전용 |
| 자동 커밋 | 기본 | 수동 |
| Repo Map | 강력 (자체 구현) | Glob/Grep 기반 |
| TUI 화려함 | 단순/실용 | 풍부 |
| 생태계 | OSS, 플러그인 적음 | MCP/Skills/Hooks |

**언제 Aider인가**: 모델 자유도, 자동 git 흐름, 로컬/저렴한 모델로 비용 통제.

---

## 참고

- 공식: https://aider.chat/
- GitHub: https://github.com/Aider-AI/aider
- 모델별 점수: https://aider.chat/docs/leaderboards/
