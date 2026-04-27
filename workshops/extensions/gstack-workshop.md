# gstack 핸즈온 워크샵

[gstack](../../docs/extensions/gstack.md) (garrytan/gstack) 은 Y Combinator CEO **Garry Tan**의 Claude Code 셋업 그대로. 30+ 개 스킬이 **CEO / Designer / Eng Manager / QA / Release / SRE** 같은 역할로 묶여 있고, 한 사이클을 다음으로 강제합니다:

```
Think → Plan → Build → Review → Test → Ship → Reflect
```

이 워크샵에서는 **간단한 웹 랜딩 페이지** (Hero + Features + CTA) 를 gstack 사이클로 만들어보면서:
- `/office-hours` 가 어떻게 아이디어를 강제 질문으로 재정의하는지
- `/autoplan` 이 CEO/Eng/Design 리뷰를 어떻게 한 번에 돌리는지
- `/design-shotgun` 이 시안 4~6개를 어떻게 만들고 비교하는지
- `/qa` 가 진짜 브라우저로 어떻게 테스트하는지
- `/ship` → `/land-and-deploy` 가 어떻게 PR-merge-deploy 까지 끌고가는지

매 단계 **결과 확인 체크포인트**가 있습니다.

> 사전 학습: [Claude Code 워크샵](../agents/claude-code-workshop.md)
> 레퍼런스: [`docs/extensions/gstack.md`](../../docs/extensions/gstack.md)

---

## 0. 사전 준비

| 항목 | 확인 / 설치 |
|---|---|
| Claude Code | `claude --version` |
| Git | `git --version` |
| Bun v1.0+ | `bun --version` (없으면 `curl -fsSL https://bun.sh/install \| bash`) |
| Node.js | `node --version` (Windows는 필수, macOS는 권장) |
| Chromium 메모리 | 빌드+테스트 같이 돌릴 메모리 4GB+ 여유 |

---

## 1. 설치 (30초)

```bash
git clone --single-branch --depth 1 \
  https://github.com/garrytan/gstack.git \
  ~/.claude/skills/gstack && \
  cd ~/.claude/skills/gstack && \
  ./setup
```

`./setup` 이:
- `~/.claude/CLAUDE.md` 에 gstack 섹션 추가
- 의존 도구 설치 (Bun 패키지)
- Playwright 브라우저 다운로드

### 팀 모드 (선택)

여러 명이 같은 repo 에서 쓸 거면:

```bash
(cd ~/.claude/skills/gstack && ./setup --team) && \
  ~/.claude/skills/gstack/bin/gstack-team-init required
```

자동 업데이트 + 공유 설정.

### Claude Code 외 다른 에이전트에 깔기

```bash
cd ~/.claude/skills/gstack && ./setup --host codex   # 또는 cursor / hermes / factory-droid
```

### ✅ 결과 확인

```bash
ls ~/.claude/skills/gstack/
# bin/  commands/  ...

ls ~/.claude/skills/gstack/commands/ | head
# office-hours.md  plan-ceo-review.md  ...

cat ~/.claude/CLAUDE.md | grep -A 5 "gstack"
# gstack 섹션 존재 확인
```

Claude Code 재시작 후:

```
> /office-hours
```

명령이 인식되면 OK. 인식 안 되면 Claude Code 한 번 더 종료/실행.

---

## 2. 빈 프로젝트 폴더

```bash
mkdir ~/Desktop/gstack-demo
cd ~/Desktop/gstack-demo
git init
echo "node_modules\ndist" > .gitignore
git add .gitignore
git commit -m "chore: init"
claude
```

---

## 3. 🧠 Think — `/office-hours`

```
> /office-hours

내 아이디어: 우리 팀 사내 도구 이름 / 슬로건 / 데모 영상 1개를 보여주는
랜딩 페이지를 만들고 싶어. 사용자가 가입 폼에 이메일 남기면 베타 알림.
```

`/office-hours` 가 6개 forcing question 을 던집니다 (실제 질문은 매번 다름):

| 영역 | 예시 질문 |
|---|---|
| 진짜 문제 | 가입자 모으는 게 진짜 목표인가, 아니면 평판 만드는 거? |
| 가설 검증 | 이메일 모은 다음 무엇을 확인할 건데? |
| 안 만들면 어떻게 되나 | 정적 HTML 한 페이지 / 노션 / 폼 서비스 로 충분하지 않나 |
| 누가 진짜 쓰나 | 유저 페르소나는? 지금 어디서 우리 못 만나? |
| 6주 뒤 | 이 페이지의 성공 기준은? |
| 구현 옵션 | 1) Vite+React 풀빌드, 2) Astro 정적, 3) 노션-임베드 — 각 effort? |

답변하면서 자기 가정이 깨지는 경험. 마지막에 **DESIGN.md** 에 합의된 디자인 doc이 저장됨.

### ✅ 결과 확인

```bash
ls
# DESIGN.md 가 생성됨

cat DESIGN.md
```

내용 예 (요약):

```markdown
# Project: Beta Landing

## Problem
- 베타 출시 전 흥미가 있는 잠재 사용자 확보
- 평판 신호: "우리는 진짜 만들고 있다"는 시각 자료

## Approach
선택: Astro 정적 사이트 + 이메일 폼 (Buttondown API)

## Success criteria
6주 뒤: 200+ 베타 가입, 그 중 30%+ 가 알파 테스트 응답

## Out of scope
- 사용자 로그인
- 결제
- 다국어 (v2)
```

---

## 4. 🧭 Plan — `/autoplan`

CEO 리뷰, 엔지니어링 리뷰, 디자인 리뷰를 **한 번에**:

```
> /autoplan
```

내부적으로:
1. `/plan-ceo-review` — 스코프 확장 / 축소 결정 (4 모드: Expansion / Selective / Hold / Reduction)
2. `/plan-eng-review` — ASCII 데이터 플로우 / 에러 path / test matrix
3. `/plan-design-review` — 디자인 dimension 0-10 점수 + AI slop 검출

**각 리뷰가 자기 영역만 평가** 하고, 전체적으로 사용자에게 **taste 결정**만 묻습니다.

> 💡 개별 리뷰만 하고 싶으면 `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review`, `/plan-devex-review` 직접 호출.

### ✅ 결과 확인

DESIGN.md 가 다음으로 채워집니다:

```bash
cat DESIGN.md
```

추가된 섹션:
- **CEO Review** — 스코프 확장 권장 (예: "이메일 받았으니 'thanks for signup' 페이지도 같이")
- **Eng Review** — 데이터 플로우 ASCII, 테스트 매트릭스, 에러 핸들링
- **Design Review** — 각 dimension 점수, 개선점

이걸 보고 동의 안 하는 부분 짚고 수정:

```
> CEO 리뷰의 thanks 페이지는 너무 oversized. 빼자.
```

---

## 5. 🎨 Design — `/design-shotgun` & `/design-html`

### 시안 4~6개 동시에 보기

```
> /design-shotgun

랜딩 페이지 hero 섹션. 텍스트: "Stop fighting your todo list."
배경 이미지 / gradient / 일러스트 등 직접적 옵션 보여줘.
```

gstack 이:
1. GPT Image API 로 4~6개 mockup 변형 생성
2. **브라우저에 비교 보드** 자동 오픈 (side-by-side)
3. 각 시안에 ✅/❌ 체크 → 답변 보고 다음 round 생성
4. **취향 학습** — 5%/주 decay 로 점차 잘 맞춰짐

### ✅ 결과 확인

브라우저에 비교 보드가 자동으로 열려야 합니다. 안 열리면:

```bash
ls ~/.gstack/sketches/<project>/
# variant-1.png  variant-2.png  ...  comparison.html
```

`comparison.html` 직접 열어 확인.

선호 시안을 골라 다음 단계로:

```
> 1번이 마음에 들어. 이걸로 가자.
```

### Mockup → 실제 코드

```
> /design-html
```

선택한 mockup을 **production HTML/CSS** 로 변환:
- Pretext 엔진으로 ~30KB 결과
- 0 dependency
- responsive (텍스트 reflow)
- 프레임워크 자동 감지 (이 워크샵에선 Astro 권장)

### ✅ 결과 확인

```bash
ls src/
# pages/index.astro  components/Hero.astro  ...

git log --oneline | head
# feat(design): hero section from shotgun variant 1
```

빌드해보기:

```bash
pnpm install
pnpm dev
# http://localhost:4321 열어서 확인
```

---

## 6. 🛠️ Build — 일반 Claude Code 흐름

implementation 자체는 gstack의 별도 스킬 없음. **그냥 Claude Code 로** 평소대로:

```
> 이메일 폼을 Buttondown API에 POST. 폼 검증 + 에러 메시지.
```

Claude Code가 코드 작성. (Superpowers 와 같이 쓰면 여기서 brainstorming → plan → execute 흐름 자동 발동.)

### ✅ 결과 확인

```bash
git log --oneline
# feat: email form with validation
# feat: buttondown api integration
```

---

## 7. 🔍 Review — `/review`

빌드 끝났으면 production-ready 감사:

```
> /review
```

`/review` 가:
1. 브랜치 diff 분석
2. **CI는 통과하지만 production에서 깨질 패턴** 식별
3. 명백한 것은 **자동 수정 + atomic commit**
4. 모호한 건 사용자 승인 요청
5. 완료 직전 fix 검증

### ✅ 결과 확인

```bash
git log --oneline | head
# fix(review): trim email before validation
# fix(review): handle 429 rate limit from buttondown
# ...

# audit 보고서 위치 (보통 PR 본문 또는 .gstack/reviews/)
ls .gstack/reviews/ 2>/dev/null
```

---

## 8. 🛡️ Security — `/cso`

배포 전 보안 감사:

```
> /cso
```

CSO 스킬 (Chief Security Officer):
1. 데이터 플로우 + trust boundary 매핑
2. **OWASP Top 10** 체크
3. **STRIDE** 위협 모델
4. 17개 false positive 제외 룰
5. 8/10+ 신뢰도 발견만 보고
6. 각 발견에 대해 **구체적 exploit 시나리오** 제공

### ✅ 결과 확인

```
[CSO Audit Report]

Threat: Stored XSS via email field
   Confidence: 9/10
   Path: src/api/subscribe.ts → buttondown 외부 호출 (안전)
        but render at /thanks 페이지에서 raw email 출력
   Exploit: 가입자가 <script>...</script> 입력 → /thanks 방문자 피해
   Fix: textContent (innerHTML 금지) 또는 server escape
```

발견된 게 있으면 fix → 다시 `/cso` 까지 clean 까지 반복.

---

## 9. 🧪 QA — `/qa`

**진짜 브라우저로** 테스트:

```
> /qa staging URL: http://localhost:4321
```

`/qa` 가:
1. headless Chromium 오픈
2. 모든 flow 클릭 (네비게이션 / 폼 / 모달)
3. 버그를 **스크린샷으로 캡처**
4. 각 버그 fix → atomic commit
5. **regression test 자동 생성** (Playwright)
6. fix 가 정말 동작하는지 재검증

### 인증된 페이지 테스트

이 데모엔 안 필요하지만, 로그인 필요한 사이트라면:

```
> /setup-browser-cookies
   (Chrome / Arc / Brave 에서 쿠키 가져옴)
> /qa
```

### ✅ 결과 확인

```bash
ls .gstack/qa-runs/<timestamp>/
# screenshots/   bug-001.png  bug-002.png
# bug-report.md
# regression-tests/  *.spec.ts

git log --oneline | head
# test(qa): regression for empty email submission
# fix(qa): show error toast on duplicate email
```

**fix 만 보지 말고 reports 한 번 읽어보세요** — gstack 이 본 시각 / 클릭 / 결과 가 다 들어 있어 디버깅에 매우 유용.

> 💡 fix 없이 issue 만 받고 싶으면 `/qa-only`

---

## 10. 📊 Performance — `/benchmark`

(선택) 성능 baseline:

```
> /benchmark
```

- 페이지 로드 시간
- Core Web Vitals (LCP / CLS / INP)
- 리소스 사이즈
- 이전 PR 대비 비교

### ✅ 결과 확인

```bash
ls .gstack/benchmarks/
# baseline.json  pr-1.json  comparison.md
```

---

## 11. 🚀 Ship — `/ship`

```
> /ship
```

`/ship` 이:
1. main 브랜치 sync
2. 모든 테스트 실행
3. 커버리지 audit (없으면 부트스트랩)
4. **`/document-release` 자동 호출** — README/docs 업데이트
5. 브랜치 push
6. PR 생성

### ✅ 결과 확인

```bash
gh pr view --web
```

PR 본문에:
- 변경 요약
- 테스트 커버리지 비교
- 업데이트된 docs 요약
- 검증 결과

---

## 12. 🎯 Land + Deploy — `/setup-deploy` & `/land-and-deploy`

### 첫 배포 환경 셋업

```
> /setup-deploy
```

자동 감지: Vercel / Netlify / Railway / Heroku / Cloudflare Pages 등. 프로덕션 URL + deploy 명령 발견 → 테스트.

### ✅ 결과 확인

```bash
cat .gstack/deploy.yaml
# platform: vercel
# url: https://gstack-demo.vercel.app
# deploy_cmd: vercel deploy --prod
```

### PR 머지 → 배포 → 검증

```
> /land-and-deploy
```

자동으로:
1. 승인된 PR merge
2. CI 통과 대기
3. 프로덕션 배포 트리거
4. 배포 health 체크 (콘솔 에러 / 페이지 실패)

### ✅ 결과 확인

```bash
gh run list --limit 1
# CI run 성공 확인

# 배포된 URL 직접 방문
open https://gstack-demo.vercel.app
```

배포된 사이트를 진짜 브라우저로 본 결과가 PR 코멘트에 자동으로 달림 (스크린샷 + 콘솔 로그).

---

## 13. 🦅 Canary — `/canary`

배포 후 1~24시간 모니터링:

```
> /canary
```

콘솔 에러 / 성능 회귀 / 페이지 실패 감지 → 알림.

### ✅ 결과 확인

```bash
ls .gstack/canary-runs/
# 모니터링 결과 + 알림 로그
```

---

## 14. 🔁 Reflect — `/retro` & `/learn`

### 주간 회고

```
> /retro
```

- 출하 history
- 사람별 breakdown
- ship 연속 일수
- 테스트 health 트렌드
- 성장 기회

`/retro global` — 모든 프로젝트 통합.

### 패턴 라이브러리

```
> /learn
```

세션 간 학습 정리:
- pattern library 검토
- outdated 패턴 제거
- 프로젝트 지식 export → 다음 세션에 자동 컨텍스트

### ✅ 결과 확인

```bash
ls ~/.gstack/learnings/
# patterns.md  styles.md  pitfalls.md

cat ~/.gstack/learnings/patterns.md
```

---

## 15. 🧠 GBrain — 영속 메모리 셋업 (선택)

여러 디바이스 / 팀 / 세션 간에 공유되는 영속 메모리:

```
> /setup-gbrain
```

3가지 경로:

| 경로 | 셋업 | 특징 |
|---|---|---|
| Supabase 기존 프로젝트 | Session Pooler URL paste | 클라우드 공유 |
| Supabase 자동 프로비전 | Personal Access Token | ~90초, 새 프로젝트 자동 생성 |
| PGLite 로컬 | 즉시 | ~30초, 머신 한정, 나중에 `--switch` 로 클라우드 이전 가능 |

### Trust 정책 (per-repo)

| 정책 | 의미 |
|---|---|
| `read-write` | 검색 + 쓰기 |
| `read-only` | 검색만 (consultant 모드) |
| `deny` | 액세스 차단 |

### ✅ 결과 확인

```
> /mcp
```

MCP 서버 목록에 `gbrain` 이 등록되어 있어야 함.

---

## 16. 🛡️ 안전 모드 — `/freeze` `/careful` `/guard`

위험한 작업 / 디버깅 시:

| 명령 | 동작 |
|---|---|
| `/careful` | `rm -rf` / `DROP TABLE` / `git push --force` / `git reset --hard` 전 경고 |
| `/freeze <dir>` | 그 디렉토리 밖 편집 금지 (디버깅 중 scope creep 방지) |
| `/guard` | 위 둘 다 |
| `/unfreeze` | freeze 해제 |

### 사용 예

```
> /freeze src/auth
> auth 모듈에서 토큰 만료 처리 버그 찾아서 고쳐줘
```

→ `src/auth/` 만 수정 가능. 다른 곳 건드리려 하면 차단됨.

---

## 17. 🤝 멀티 에이전트 — `/pair-agent`

Claude Code + OpenClaw + Hermes 같은 여러 에이전트가 **같은 브라우저** 공유:

```
> /pair-agent
```

- gstack 브라우저 launch
- one-time key → session token 교환
- 에이전트마다 격리된 탭
- 원격 에이전트라면 ngrok 자동
- 탭 격리 / rate limiting / activity attribution 강제

### ✅ 결과 확인

```bash
# 출력된 setup 명령을 다른 에이전트(OpenClaw 등)에서 실행
# 그러면 같은 브라우저의 별도 탭에 attach
```

---

## 18. 📈 분석 / 모델 비교

### 사용 통계

```bash
gstack-analytics
```

스킬별 사용 빈도, 시간, 성공률. **로컬 only**.

### 크로스 모델 비교

```bash
gstack-model-benchmark --prompt "<prompt>" --dry-run
```

같은 프롬프트를 Claude / GPT (via Codex) / Gemini 에 → latency / 토큰 / 비용 / (옵션) 품질 점수 비교.

---

## 19. 🔄 업데이트

```
> /gstack-upgrade
```

자동 감지 (글로벌 vs vendored), 변경 요약 출력.

---

## 20. 트러블슈팅

### 명령이 인식 안 됨

```bash
# 설치 잘 됐는지
ls ~/.claude/skills/gstack/commands/ | wc -l   # 30+ 이어야 함

# CLAUDE.md 에 gstack 섹션 있는지
grep -A 3 gstack ~/.claude/CLAUDE.md
```

없으면 setup 다시:

```bash
cd ~/.claude/skills/gstack && ./setup
```

Claude Code 재시작.

### 브라우저 안 뜸

Playwright 브라우저 미설치:

```bash
cd ~/.claude/skills/gstack
bun install
bunx playwright install chromium
```

### Bun 없음

```bash
curl -fsSL https://bun.sh/install | bash
```

### `gstack-analytics` 못 찾음

```bash
ls ~/.claude/skills/gstack/bin/
# gstack-analytics 가 있어야 함

# PATH 에 추가
export PATH="$HOME/.claude/skills/gstack/bin:$PATH"
```

---

## 21. 전체 흐름 요약

```
[설치] git clone + ./setup
   ↓
🧠 /office-hours       → DESIGN.md (forcing questions)
   ↓
🧭 /autoplan           → CEO + Eng + Design 리뷰 결과 추가
   ↓
🎨 /design-shotgun     → 시안 4~6개 비교 보드
   ↓ (시안 선택)
🎨 /design-html        → production HTML/CSS
   ↓
🛠️  Build (일반 Claude Code)
   ↓
🔍 /review             → production 패턴 audit + auto fix
   ↓
🛡️ /cso               → OWASP + STRIDE 보안 감사
   ↓
🧪 /qa                 → 진짜 브라우저 테스트 + regression
   ↓
📊 /benchmark         → 성능 비교
   ↓
🚀 /ship              → tests + docs + PR
   ↓
🎯 /land-and-deploy    → merge → CI → deploy → health
   ↓
🦅 /canary            → 24h 모니터링
   ↓
🔁 /retro / /learn    → 회고 + 패턴 학습 → 다음 사이클로
```

수고하셨습니다 🎉

---

## 22. 다른 스택과 함께

| 조합 | 효과 |
|---|---|
| gstack + [Superpowers](./superpowers-workshop.md) | 역할 (gstack) + 디시플린 (Superpowers). 추천. |
| gstack + [GSD](./gsd-workshop.md) | 역할 (gstack) + 단계 컨텍스트 (GSD). 큰 프로젝트. |
| gstack + Superpowers + GSD | 풀스택. [DEV.to 가이드](https://dev.to/imaginex/a-claude-code-skills-stack-how-to-combine-superpowers-gstack-and-gsd-without-the-chaos-44b3) |

---

## 23. 더 보기

- 레퍼런스: [`docs/extensions/gstack.md`](../../docs/extensions/gstack.md)
- 메인 repo: https://github.com/garrytan/gstack
- 공식 사이트: https://gstacks.org/
- HN 토론: https://news.ycombinator.com/item?id=47355173
- 튜토리얼: https://www.sitepoint.com/gstack-garry-tan-claude-code/
