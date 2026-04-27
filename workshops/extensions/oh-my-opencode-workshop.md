# oh-my-opencode (oh-my-openagent) 핸즈온 워크샵

[oh-my-opencode](../../docs/extensions/oh-my-opencode.md) 는 [OpenCode](../../docs/agents/opencode.md) 위에 얹는 **에이전트 오케스트레이션 하니스**. v3+ 부터 `oh-my-openagent` 로 리브랜딩 진행 중. **v3.17.5** (2026-04-24 기준 최신).

핵심 가치: **다중 모델 동시 사용** (Claude / Kimi / GLM / GPT / Gemini) + **6개 빌트인 에이전트** 가 협력.

```
Sisyphus (오케스트레이터)
   ├─ Hephaestus (자율 실행자, GPT-5.4 베이스)
   ├─ Prometheus (전략 플래너, 인터뷰 진행)
   ├─ Oracle (아키텍처/디버깅)
   ├─ Librarian (문서/코드 검색)
   └─ Explore (빠른 grep)
```

이 워크샵에서는 **간단한 REST API 서버** (Hono + SQLite + 인증) 를 oh-my-opencode 로 만들어보면서:
- `ultrawork` (`ulw`) 가 어떻게 모든 에이전트를 동원하는지
- `/start-work` 의 Prometheus 가 어떻게 인터뷰하는지
- `/init-deep` 이 hierarchical AGENTS.md 를 어떻게 만드는지
- 빌트인 MCP (Exa / Context7 / Grep.app) 가 어떻게 외부 정보를 가져오는지
- Claude Code 호환 레이어로 superpowers / gstack 같은 기존 스킬을 그대로 재사용하는 방식

---

## 0. 사전 준비

| 항목 | 확인 |
|---|---|
| Bun 또는 Node 18+ | `bun --version` / `node --version` |
| OpenCode 본체 | `opencode --version` |
| 모델 API 키 (1개 이상) | Claude / OpenAI / Kimi / GLM 중 |

OpenCode 가 없으면:

```bash
curl -fsSL https://opencode.ai/install | bash
opencode --version
```

API 키 환경변수 (있는 거 하나라도):

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export MOONSHOT_API_KEY="..."   # Kimi
export ZHIPU_API_KEY="..."      # GLM
```

---

## 1. 설치

```bash
# npm
npm install -g oh-my-opencode

# 또는 bunx (인스턴트)
bunx oh-my-opencode

oh-my-opencode --version   # v3.17.5+
```

### OpenCode 에 plugin 등록

`~/.config/opencode/opencode.json`:

```json
{
  "plugins": ["oh-my-openagent"]
}
```

> 레거시 `"oh-my-opencode"` 는 지금도 로딩되지만 warning 출력. 새 키 권장.

### ✅ 결과 확인

```bash
# 진단 명령 — 빌트인
bunx oh-my-opencode doctor
```

다음 같은 출력이 정상:

```
✅ OpenCode found (v0.x.x)
✅ Bun runtime
✅ ANTHROPIC_API_KEY set
⚠️ MOONSHOT_API_KEY missing  (Kimi 사용 안 하면 OK)
✅ Plugin loaded: oh-my-openagent
✅ MCP servers: exa, context7, grep.app
```

---

## 2. 빈 프로젝트 폴더

```bash
mkdir ~/Desktop/omo-demo
cd ~/Desktop/omo-demo
git init
echo "node_modules\ndb.sqlite" > .gitignore
git add .gitignore
git commit -m "chore: init"
```

OpenCode 실행:

```bash
opencode
```

---

## 3. 첫 명령 — `/start-work` (Prometheus 플래너)

```
> /start-work

Hono 프레임워크로 REST API 만들고 싶어.
- /todos 의 CRUD
- 사용자 인증 (JWT)
- SQLite로 영속
- TypeScript
```

`/start-work` 가 **Prometheus** 에이전트를 발동. 인터뷰 시작:

| 영역 | 예시 질문 |
|---|---|
| 인증 범위 | 회원가입까지 우리가? OAuth provider만? |
| 토큰 수명 | access token만? refresh? 만료? |
| DB 마이그레이션 | drizzle / prisma / 직접 SQL? |
| 에러 응답 | RFC 7807 Problem JSON? 직접 정의? |
| 테스트 | 어디까지? E2E 만? unit + integration? |

Prometheus 가 답변을 모아 **scope 정의서** 를 만듭니다.

### ✅ 결과 확인

```bash
ls
# AGENTS.md  PLAN.md  같은 파일이 생성될 수 있음 (버전마다 위치 다름)

cat AGENTS.md   # 또는 PLAN.md
```

내용 예:

```markdown
# Project: Todo API

## Scope (Prometheus 정리)
- Hono + TypeScript + Bun
- SQLite (better-sqlite3) + drizzle ORM
- JWT 인증 (access 1h + refresh 7d)
- Problem JSON 에러
- vitest unit + integration

## Out of scope
- OAuth 외부 provider (v2)
- Rate limiting (v2)
- 관리자 권한
```

---

## 4. 깊은 컨텍스트 — `/init-deep`

기존 프로젝트라면 더 유용하지만, 작은 프로젝트도 시작 시 한 번 돌리면 좋음.

```
> /init-deep
```

`/init-deep` 이 **계층적 AGENTS.md** 를 생성:

```
.
├── AGENTS.md          (프로젝트 전체)
├── src/
│   ├── AGENTS.md      (src 디렉토리 컨벤션)
│   ├── api/
│   │   └── AGENTS.md  (API 라우트 패턴)
│   └── db/
│       └── AGENTS.md  (마이그레이션 / 스키마 규칙)
└── tests/
    └── AGENTS.md      (테스트 패턴)
```

각 디렉토리의 AGENTS.md는 **그 디렉토리 작업 시에만 자동 로딩** — 메인 컨텍스트 절약.

### ✅ 결과 확인

```bash
find . -name AGENTS.md
# ./AGENTS.md
# ./src/AGENTS.md
# ./src/api/AGENTS.md
# ...

# 하나 열어보기
cat src/api/AGENTS.md
```

내용 예:

```markdown
# src/api 컨벤션

- 모든 라우트는 `c.json()` 로 응답
- 에러는 throw → 글로벌 핸들러가 Problem JSON 변환
- 인증 필요 라우트는 `authMiddleware` 적용
- 입력 검증: zod schema 먼저 (라우트 핸들러 진입 전)
```

---

## 5. 메인 명령 — `ultrawork` (`ulw`)

scope 가 정해졌으면 풀 사이클:

```
> ultrawork
```

또는 단축:

```
> ulw
```

이게 **모든 에이전트를 발동**:

```
[Sisyphus 시작]
  ↓ 작업 분석, 분할
  ├─ [Hephaestus] src/db/schema.ts 작성 (자율)
  ├─ [Hephaestus] src/api/todos.ts 작성 (자율, 병렬)
  └─ [Oracle] 인증 미들웨어 설계 검토
        ↓
  [Librarian] Hono JWT 미들웨어 공식 문서 확인
        ↓
  [Hephaestus] 구현 + 테스트
        ↓
  [Sisyphus] 통합 + 검증 → 완료
```

각 단계 사이에 **fresh context 분기** — 큰 작업도 메인 컨텍스트 보호.

### ✅ 결과 확인

```bash
# 코드 생성 확인
ls src/
# api/  db/  middleware/  index.ts

# git history (각 에이전트가 자기 단위로 commit)
git log --oneline
# feat(db): drizzle schema for users + todos
# feat(api): todos CRUD routes
# feat(auth): JWT middleware
# test: integration coverage for todos

# 빌드 / 실행
bun install
bun run src/index.ts
# http://localhost:3000 띄움

# curl 테스트
curl -X POST http://localhost:3000/api/auth/signup \
  -H 'Content-Type: application/json' \
  -d '{"email":"a@b.c","password":"test1234"}'
```

---

## 6. 에이전트 직접 호출하기

`ultrawork` 대신 특정 에이전트만 부르고 싶을 때.

### Oracle — 아키텍처 / 디버깅

```
> @oracle 인증 토큰을 cookie로 보낼지 Authorization 헤더로 보낼지 결정해줘. trade-off 분석.
```

Oracle 이 (Claude Sonnet/Opus 또는 설정 모델):
- 두 옵션 비교
- 보안 / UX / CSRF / CORS 영향
- 추천 + 근거

### Librarian — 문서 / 코드 검색

```
> @librarian Hono 의 c.var.<key> 는 타입 추론 어떻게 되나? 공식 문서 + 실제 사용 예 모아줘
```

Librarian 이 빌트인 MCP를 활용:
- **Context7** 으로 Hono 공식 문서
- **Grep.app** 으로 GitHub 의 실제 사용 예
- **Exa** 로 보충 자료

### Explore — 빠른 grep

```
> @explore 이 프로젝트에서 jwt 토큰 검증 어디서 이뤄지지?
```

가벼운 grep만 — 메인 컨텍스트 안 잡아먹음.

### Hephaestus — 자율 실행자

```
> @hephaestus 모든 라우트에 input validation 적용해줘. 각자 알아서 schema 만들고 적용
```

scope 가 명확하면 던지고 끝. GPT-5.4 베이스 (또는 설정에 따라 변경).

### ✅ 결과 확인 (각 에이전트별로)

각 호출 후:

```bash
git log --oneline | head
# 각 에이전트의 작업이 별도 commit으로 남아 있어야 함
```

---

## 7. 빌트인 MCP 직접 사용

oh-my-opencode 가 자동으로 등록한 MCP 3종은 사용자가 직접 호출도 가능.

### Exa (웹 검색)

```
> Exa 로 "Hono JWT middleware 2026" 검색해줘
```

### Context7 (공식 문서)

```
> Context7 에서 "drizzle-orm" 공식 문서 가져와
```

### Grep.app (GitHub 코드 검색)

```
> Grep.app 으로 GitHub 에서 "honoFactory.basePath" 사용 예 5개 찾아줘
```

### ✅ 결과 확인

```
> /mcp
```

활성 MCP 목록과 호출 통계 확인. 다음이 보여야 함:
- `exa` (웹 검색)
- `context7` (문서)
- `grep.app` (코드)

---

## 8. Claude Code 호환 레이어

oh-my-opencode 의 핵심 셀링 포인트: **Claude Code 의 hook / command / skill / MCP / plugin 이 그대로 동작**.

### 기존 Claude Code 자산을 OpenCode 에 그대로

```bash
# Claude Code에 깐 superpowers / gstack 를
# OpenCode 안에서도 사용
ls ~/.claude/skills/
# gstack/  superpowers (plugin)/  ...
```

oh-my-opencode 가 이걸 자동으로 로드. 별도 설정 없이.

### 검증

```
> /office-hours   (gstack 명령)
> /brainstorm     (superpowers 명령)
```

OpenCode 안에서 인식되면 호환 레이어 OK.

### ✅ 결과 확인

```bash
bunx oh-my-opencode doctor
# Claude Code compat layer: ✅
# Detected skills: superpowers, gstack
```

---

## 9. 자기 참조 루프 — `/ulw-loop` (Ralph Loop)

복잡한 리팩터를 **자기가 자기를 호출하면서 점진**:

```
> /ulw-loop

목표: 모든 에러 응답을 Problem JSON 으로 통일.
중단 조건: 모든 라우트가 통일됨 + 테스트 통과.
```

루프가 매 iteration:
1. 현재 상태 분석
2. 다음 한 스텝 결정
3. 실행 → commit
4. 종료 조건 체크
5. 안 끝났으면 다시 1

### ✅ 결과 확인

```bash
git log --oneline | grep -i "loop"
# loop iter 1: convert /api/todos errors
# loop iter 2: convert /api/auth errors
# loop iter 3: add global error handler
# loop iter 4: all routes verified, exit
```

위험: 무한 루프 가능 → 종료 조건 명확히.

---

## 10. 모델 라우팅 (다중 모델 동시)

Sisyphus 가 작업별로 모델을 자동 선택. 또는 명시적으로:

```
> /model

[다음 중 선택]
1. claude-opus-4.7    (큰 추론)
2. claude-sonnet-4.6  (일반)
3. kimi-k2.5          (저렴, 컨텍스트 큼)
4. glm-5              (저렴, 중국어 강함)
5. gpt-5.4            (Hephaestus 기본)
```

### Configuration

`~/.config/opencode/oh-my-openagent.jsonc`:

```jsonc
{
  // 에이전트별 모델 선호도
  "agents": {
    "sisyphus":   { "model": "claude-opus-4-7" },
    "hephaestus": { "model": "gpt-5-4" },
    "prometheus": { "model": "claude-sonnet-4-6" },
    "oracle":     { "model": "claude-opus-4-7" },
    "librarian":  { "model": "kimi-k2-5" },  // 컨텍스트 큰 게 유리
    "explore":    { "model": "glm-5" }       // 저렴한 게 적당
  },
  // 비용 한도 (선택)
  "budget": { "daily_usd": 10 }
}
```

JSONC 라서 주석 / trailing comma 됨.

### ✅ 결과 확인

```bash
cat ~/.config/opencode/oh-my-openagent.jsonc

# 각 에이전트가 설정된 모델로 도는지
> @oracle 그냥 ping
# 응답 끝에 "via claude-opus-4-7" 같은 표시
```

---

## 11. 백그라운드 / 병렬 에이전트

```
> @hephaestus background: 모든 함수에 jsdoc 추가
> @oracle 인증 흐름 다이어그램 그려줘
```

위 둘이 **동시에** 백그라운드로 진행. 메인 세션은 자유.

### ✅ 결과 확인

```
> /jobs
```

활성 백그라운드 작업 목록 + 진행률. 끝나면 알림.

---

## 12. 트러블슈팅

### 진단 한 방

```bash
bunx oh-my-opencode doctor
```

거의 모든 문제를 잡아냄.

### 명령 인식 안 됨

```bash
# OpenCode plugin 로딩 확인
cat ~/.config/opencode/opencode.json
# "plugins": ["oh-my-openagent"] 가 있어야 함

# 레거시 fallback
# "plugins": ["oh-my-opencode"]  도 동작은 함 (warning)
```

### MCP 가 안 등록됨

```bash
# 빌트인 MCP 강제 재설치
bunx oh-my-opencode --reinstall-mcps
```

### Claude Code 호환 레이어 무시됨

```bash
# 호환 레이어 명시적 활성
# oh-my-openagent.jsonc:
{ "compat": { "claude_code": true } }
```

### 토큰가 폭주

```bash
# 비용 한도 강제
{ "budget": { "daily_usd": 5 } }

# 또는 저렴한 모델로 라우팅
{ "agents": { "*": { "model": "kimi-k2-5" } } }
```

---

## 13. 변형 / 포크 비교

| 패키지 | 특징 | 추천 상황 |
|---|---|---|
| [`opensoft/oh-my-opencode`](https://github.com/opensoft/oh-my-opencode) | 원조, 풀팩 | 기본 추천 |
| [`code-yeongyu/oh-my-openagent`](https://github.com/code-yeongyu/oh-my-openagent) | 리브랜딩 (omo) | 최신 dev 따라가고 싶을 때 |
| [`alvinunreal/oh-my-opencode-slim`](https://github.com/alvinunreal/oh-my-opencode-slim) | 토큰 소비 절감 fork | 비용 민감 |
| [`WilliamJudge94/oh-my-opencode-dashboard`](https://github.com/WilliamJudge94/oh-my-opencode-dashboard) | 에이전트 추적 대시보드 | 멀티 에이전트 모니터링 |

전환 방법:

```bash
npm uninstall -g oh-my-opencode
npm install -g <원하는-패키지>
```

설정 파일은 호환 (이름만 다를 뿐).

---

## 14. 전체 흐름 요약

```
[설치] npm i -g oh-my-opencode + opencode.json plugins 등록
   ↓
[OpenCode 실행]
   ↓
/start-work        → Prometheus 인터뷰 → AGENTS.md / PLAN
   ↓
/init-deep         → 디렉토리별 AGENTS.md 생성 (선택)
   ↓
ultrawork (ulw)    → 모든 에이전트 동원해서 풀 사이클
   ├─ Sisyphus 분할
   ├─ Hephaestus 자율 실행 (병렬)
   ├─ Oracle 검증
   └─ Librarian/Explore 정보 수집
   ↓
@oracle / @hephaestus / @librarian / @explore  (개별 호출)
   ↓
/ulw-loop          → 점진 리팩터 (자기 참조)
   ↓
/mcp 로 외부 데이터 (Exa/Context7/Grep.app)
   ↓
[Claude Code 호환 레이어]
  → 기존 superpowers / gstack 자산 그대로 활용
```

---

## 15. 다른 스택과 함께

| 조합 | 효과 |
|---|---|
| oh-my-opencode + [Superpowers](./superpowers-workshop.md) | OpenCode 위에서 디시플린까지 |
| oh-my-opencode + [gstack](./gstack-workshop.md) | OpenCode 위에서 풀사이클 역할 |
| oh-my-opencode + [GSD](./gsd-workshop.md) | OpenCode 위에서 단계 관리 (rokicool/gsd-opencode 도 참고) |

OpenCode 의 **다중 모델 + 백그라운드 에이전트** 강점 + 위 스킬팩의 **방법론** 결합 가능.

---

## 16. 더 보기

- 레퍼런스: [`docs/extensions/oh-my-opencode.md`](../../docs/extensions/oh-my-opencode.md)
- 본체 OpenCode: [`docs/agents/opencode.md`](../../docs/agents/opencode.md)
- 메인 (원조): https://github.com/opensoft/oh-my-opencode
- 리브랜딩: https://github.com/code-yeongyu/oh-my-openagent
- Slim fork: https://github.com/alvinunreal/oh-my-opencode-slim
- Awesome 모음: https://github.com/awesome-opencode/awesome-opencode
