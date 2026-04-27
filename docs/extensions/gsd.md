# GSD (Get Shit Done)

[gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) — TÂCHES가 만든 **메타 프롬프팅 / 컨텍스트 엔지니어링 / 스펙 드리븐 개발** 시스템. Claude Code를 비롯해 12개 런타임을 지원합니다.

48K~50K+ stars 의 높은 인지도. 큰 작업을 오래 끌 때 **컨텍스트 부패(context rot)** 를 막는 데 초점.

---

## 핵심 아이디어

LLM은 컨텍스트가 채워질수록 품질이 떨어집니다. GSD는 작업을 단계별 마크다운 아티팩트로 영속화해서 매 작업이 **신선한 컨텍스트**로 시작하게 만듭니다.

```
Initialize → Discuss → Plan → Execute → Verify → Ship
   ↓          ↓         ↓        ↓         ↓        ↓
 .planning/ 디렉토리에 각 단계의 markdown 산출물이 쌓임
```

각 phase의 산출물이 다음 phase의 컨텍스트가 되어, **메인 컨텍스트는 30~40% 사용률**로 유지된다고 함.

---

## 지원 런타임

Claude Code, OpenCode, Gemini CLI, Kilo, Codex, Copilot, Cursor, Windsurf, Antigravity, Augment, Trae, Cline — 한 인스톨러로 12개 모두 커버.

---

## 설치

```bash
# 가장 간단
npx get-shit-done-cc@latest

# 토큰 절감 모드
npx get-shit-done-cc@latest --minimal
```

글로벌 / 프로젝트 단위 설치 선택 가능.

---

## 주요 슬래시 커맨드

| 커맨드 | 용도 |
|---|---|
| `/gsd-new-project` | 프로젝트 전체 초기화 |
| `/gsd-discuss-phase N` | 구현 결정 캡처 |
| `/gsd-plan-phase N` | 리서치 + 검증 가능한 계획 |
| `/gsd-execute-phase N` | 병렬 wave로 태스크 실행 |
| `/gsd-verify-work N` | 수동 UAT + 자동 진단 |
| `/gsd-ship N` | 검증 끝난 work에서 PR 생성 |
| `/gsd-quick` | 계획 없이 즉석 작업 |
| `/gsd-graphify` | 계획 단계 지식 그래프 통합 (v1.36+) |
| `/gsd-help` | 커맨드 디스커버리 |

`--tdd` 플래그로 TDD 파이프라인 모드.

---

## 왜 동작하나

- **XML 구조 plan** — Claude의 instruction-following 최적화
- **멀티 에이전트 오케스트레이션** — 메인 컨텍스트 보호
- **태스크당 atomic git commit** — bisect 가능, 정밀한 히스토리
- **병렬 wave 실행** — 독립 plan의 처리량 극대화

---

## superpowers / gstack 와의 관계

세 도구 모두 Claude Code 위 레이어지만 초점이 다릅니다.

| 도구 | 초점 | 강점 |
|---|---|---|
| [Superpowers](./superpowers.md) | **디시플린** (TDD, 디버깅 절차) | 모든 작업에 일관된 방법론 |
| [gstack](./gstack.md) | **역할 기반** 워크플로우 (CEO/Eng/QA) | 풀-사이클 SaaS 개발 |
| GSD | **단계별 컨텍스트 관리** | 장기 / 큰 작업의 품질 유지 |

세 개를 같이 쓰는 [실용 가이드](https://dev.to/imaginex/a-claude-code-skills-stack-how-to-combine-superpowers-gstack-and-gsd-without-the-chaos-44b3) 도 있음.

---

## 라이선스

MIT.

---

## 참고

- 메인: https://github.com/gsd-build/get-shit-done
- v2 (Pi SDK 기반 단독 CLI): https://github.com/gsd-build/gsd-2
- OpenCode 포팅: https://github.com/rokicool/gsd-opencode
- 워크플로우 분석: https://www.codecentric.de/en/knowledge-hub/blog/the-anatomy-of-claude-code-workflows-turning-slash-commands-into-an-ai-development-system
