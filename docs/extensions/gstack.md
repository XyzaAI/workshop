# gstack

[garrytan/gstack](https://github.com/garrytan/gstack) — Y Combinator CEO **Garry Tan**이 직접 쓰는 Claude Code 셋업을 그대로 공개한 오픈소스. Claude Code 한 명을 **가상의 개발팀**으로 변신시킵니다.

23개의 의견 강한(opinionated) 도구가 CEO / Designer / Eng Manager / Release Manager / Doc Engineer / QA 같은 **역할** 단위로 쪼개져 있고, 각 역할이 슬래시 커맨드로 노출됩니다.

GitHub 16K+ stars, 1.8K+ forks. Garry Tan은 본인 워크플로우로 **2013년 대비 810배 빠른 출하 속도**를 보고했고, YC 풀타임을 하면서 60일에 3개 production 서비스 + 40+ 기능을 출시했다고 함.

---

## 철학

다음 사이클을 강제합니다:

```
Think → Plan → Build → Review → Test → Ship → Reflect
```

각 단계가 다음 단계로 자연스럽게 연결되어 **개발 파이프라인 갭**이 안 생기게 함.

---

## 설치

```bash
git clone --single-branch --depth 1 \
  https://github.com/garrytan/gstack.git \
  ~/.claude/skills/gstack && \
  cd ~/.claude/skills/gstack && ./setup
```

Claude Code skills 디렉토리에 들어가서 setup 한 줄. 30초.

---

## 주요 스킬 / 커맨드

### 기획 & 디스커버리

| 커맨드 | 역할 |
|---|---|
| `/office-hours` | 제품 아이디어를 강제 질문으로 재정의 |
| `/plan-ceo-review` | 전략적 스코프 평가 (4 모드) |
| `/plan-eng-review` | 아키텍처 + 테스트 계획 |
| `/autoplan` | 위 리뷰들을 자동 일괄 실행 |

### 디자인

| 커맨드 | 역할 |
|---|---|
| `/design-consultation` | 디자인 시스템 0부터 설계 |
| `/design-shotgun` | 시안 4~6개 생성 후 반복 개선 |
| `/design-html` | 목업 → 프로덕션 HTML/CSS |

### 코드 & 품질

| 커맨드 | 역할 |
|---|---|
| `/review` | 프로덕션 레디 코드 감사 |
| `/qa` | 브라우저 테스트 + 자동 수정 |
| `/cso` | 보안 감사 (OWASP + STRIDE) |
| `/ship` | 자동 PR + 배포 |

### 도구

| 커맨드 | 역할 |
|---|---|
| `/browse` | AI가 제어하는 실제 Chromium (100~200ms 응답) |
| `/investigate` | 근본 원인 디버깅 |
| `/canary` | 배포 후 모니터링 |
| `/retro` | 주간 회고 |

---

## 핵심 차별 기능

### `/browse` — 빠르고 영속한 Chromium

평범한 Playwright/Puppeteer는 매 호출마다 브라우저 띄우느라 느림. gstack은 **상시 떠있는 세션**을 유지해 명령당 100~200ms.
- 로그인 상태 유지
- 페이지 이동 / 스크린샷 / 레이아웃 검증
- 콘솔 에러 캡처

### GBrain — 영속 메모리

세션 간에 유지되는 지식 베이스. 에이전트가 **실제로 기억하는** 메모리.

### 풀 사이클 커버

기획 → 아키텍처 → 코드 → 리뷰 → QA → 배포 → 회고 — 코딩 단계만이 아니라 **소프트웨어 회사 워크플로우 전체**를 모델링.

---

## 누가 쓰면 좋나

- 코드를 직접 짜는 **technical founder**
- 처음 Claude Code 쓰는데 구조가 필요한 사람
- 리뷰/QA 엄격함이 필요한 **tech lead**
- 10개 이상 병렬 sprint 굴리는 팀

---

## 다른 확장과의 관계

| 도구 | 초점 |
|---|---|
| [Superpowers](./superpowers.md) | 디시플린 (TDD/디버깅 절차) |
| [GSD](./gsd.md) | 단계별 컨텍스트 관리 |
| **gstack** | **역할 기반 풀 사이클 워크플로우** |

세 개를 충돌 없이 같이 쓰는 [가이드 (DEV.to)](https://dev.to/imaginex/a-claude-code-skills-stack-how-to-combine-superpowers-gstack-and-gsd-without-the-chaos-44b3).

---

## 라이선스

MIT.

---

## 참고

- 메인: https://github.com/garrytan/gstack
- 공식 사이트: https://gstacks.org/
- HN 토론: https://news.ycombinator.com/item?id=47355173
- 튜토리얼: https://www.sitepoint.com/gstack-garry-tan-claude-code/
