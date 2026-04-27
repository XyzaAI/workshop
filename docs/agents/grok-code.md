# Grok Code

xAI의 코딩 특화 모델 `grok-code-fast` 를 활용하는 에이전트. **빠르고 저렴**한 게 핵심.

---

## 핵심 특징

- **모델**: `grok-code-fast-1` (코딩 전용 fast 모델)
- **강점**: 응답 속도 빠르고, 토큰당 가격 저렴
- **컨텍스트**: 256K
- **인증**: xAI API Key

---

## 어디서 쓰나

`grok-code` 라는 단독 CLI보단, **다른 에이전트의 백엔드 모델로 끼워서** 쓰는 경우가 많습니다.

지원 클라이언트:
- **Cursor** — 모델 선택에서 직접 가능
- **Aider** — `aider --model openai/grok-code-fast-1 --openai-api-base https://api.x.ai/v1`
- **OpenCode/Crush** — provider 설정에 xAI 추가
- **API 직접 호출** — OpenAI 호환 엔드포인트

---

## API Key 발급

[xAI Console](https://console.x.ai/) → API Keys → Create.

```bash
export XAI_API_KEY="xai-..."
# OpenAI 호환 모드라면
export OPENAI_API_KEY="$XAI_API_KEY"
export OPENAI_BASE_URL="https://api.x.ai/v1"
```

---

## 빠른 모델이 빛나는 작업

- 대량의 작은 편집 (이름 일괄 변경, 보일러플레이트 생성)
- TDD 루프 (테스트 → 실패 → 수정 → 통과 반복)
- 자동완성 / 인라인 제안
- 빌드/린트 에러 자동 수정

반대로 **깊은 추론, 아키텍처 설계, 보안 리뷰**는 Opus / o3 / Gemini Pro 같은 무거운 모델이 더 낫습니다.

---

## 가격 참고

(시점에 따라 변동, 공식 가격표 확인 필수)

`grok-code-fast` 는 Claude Sonnet/Haiku와 비슷하거나 더 저렴한 수준의 토큰가를 책정해 둠. 자주 쓰는 워크플로우(특히 Aider 같은 거)에서 비용 부담을 크게 낮출 수 있습니다.

---

## 참고

- xAI API Docs: https://docs.x.ai/
- Console: https://console.x.ai/
