# Codex 워크샵

OpenAI 계정 생성부터 요금제 등록, Codex CLI 설치(mac/windows), 실습 리포지토리 클론, Hello World 실행까지 진행합니다.

> 사전 준비: **GitHub 워크샵**을 먼저 완료하여 Git 설치 / SSH 설정 / 터미널 사용이 가능해야 합니다.

---

## 0. 이 워크샵에서 할 일

1. OpenAI 계정 만들기
2. 요금제(결제수단) 등록
3. Codex CLI 설치 (macOS / Windows)
4. 로그인
5. 실습 리포지토리 `workshop-codex` 클론
6. Codex로 Hello World 만들어보기

---

## 1. OpenAI 계정 만들기

1. [https://chatgpt.com](https://chatgpt.com) 또는 [https://platform.openai.com](https://platform.openai.com) 접속
2. 우측 상단 `Sign up` 클릭
3. 이메일 / Google / Microsoft / Apple 계정 중 선택해 가입
4. 이메일 인증 & 전화번호 인증 완료
5. 이름 · 생년월일 입력 후 계정 생성

---

## 2. 요금제 등록 (ChatGPT Plus 이상 필요)

Codex CLI는 **ChatGPT 로그인 방식**으로 사용하며, **Plus / Pro / Business / Enterprise** 플랜이 필요합니다.

### 2-1. 플랜 가입

1. [https://chatgpt.com](https://chatgpt.com) 로그인
2. 좌측 하단 프로필 → `Upgrade plan`
3. **Plus ($20/월)** 이상 선택
4. 결제수단(카드) 등록 → 구독 완료

> API Key 방식으로도 사용 가능하지만, 이 워크샵은 **ChatGPT 로그인 방식** 기준으로 진행합니다.

### 2-2. 가입 확인

- 프로필 → `My plan` 에서 Plus 이상 표시되면 OK

---

## 3. Codex CLI 설치

### 3-1. 사전 요구사항 — Node.js 설치

Codex CLI는 Node.js 기반입니다. (18 이상 권장)

→ [Node.js 설치 가이드](./docs/nodejs.md) 를 먼저 완료하세요.
(macOS는 Homebrew가 필요합니다 → [Homebrew 설치 가이드](./docs/brew.md))

### 3-2. Codex CLI 설치 (공통)

터미널(macOS Terminal / Windows Git Bash)에서:

```bash
npm install -g @openai/codex
```

설치 확인:

```bash
codex --version
```

> 권한 오류가 난다면 macOS는 `sudo npm install -g @openai/codex`, Windows는 Git Bash를 **관리자 권한**으로 실행하세요.

---

## 4. Codex 로그인

```bash
codex login
```

- 브라우저가 자동으로 열립니다.
- ChatGPT 계정으로 로그인 → 권한 허용 클릭
- 터미널에 `Login successful` 메시지가 뜨면 완료

로그인 상태 확인:

```bash
codex whoami
```

---

## 5. 실습 리포지토리 클론

`workshop-codex` 저장소를 받아 실습합니다.

### 5-1. 작업 폴더로 이동

```bash
cd ~/Desktop
```

### 5-2. 클론

```bash
git clone git@github.com:<org>/workshop-codex.git
cd workshop-codex
```

> `<org>` 는 강사가 안내하는 조직/계정명으로 바꿔주세요.
> 초대가 필요한 저장소라면 **GitHub 워크샵 5번** 과정대로 초대를 먼저 수락해야 합니다.

### 5-3. 폴더 구조 확인

```bash
ls
```

---

## 6. 실습 — Hello World 만들기

### 6-1. Codex 실행

저장소 폴더 안에서:

```bash
codex
```

> 대화형 프롬프트가 열립니다. 여기서 자연어로 Codex에게 지시할 수 있습니다.

### 6-2. 첫 번째 요청

프롬프트에 다음과 같이 입력:

```
hello.js 파일을 만들어서 "Hello, Codex!" 를 출력하도록 해줘.
```

Codex가 파일 생성을 제안하면:
- 변경사항 확인 → `y` (승인) 입력

### 6-3. 결과 실행

Codex 세션을 잠시 빠져나와(또는 새 터미널에서):

```bash
node hello.js
```

출력:

```
Hello, Codex!
```

🎉 축하합니다 — 첫 Codex 실습 성공입니다.

---

## 7. 한 걸음 더 — 연습 과제

Codex 프롬프트에 아래를 순서대로 시도해 보세요.

1. `hello.js` 가 이름을 인자로 받아 `Hello, <이름>!` 을 출력하도록 바꿔줘.
2. 같은 기능을 하는 `hello.py` 도 만들어줘.
3. `README.md` 에 실행 방법을 적어줘.

각 단계마다 결과를 실행해 확인:

```bash
node hello.js Kang
python3 hello.py Kang
```

---

## 8. 자주 쓰는 Codex 명령어

| 명령어 | 설명 |
|---|---|
| `codex` | 대화형 세션 시작 |
| `codex "프롬프트"` | 한 번만 실행 |
| `codex login` / `logout` | 로그인 / 로그아웃 |
| `codex whoami` | 현재 로그인 계정 확인 |
| `codex --help` | 전체 옵션 보기 |

세션 안에서:
- `/help` — 내부 명령어 보기
- `Ctrl + C` — 세션 종료

---

## 9. 전체 흐름 요약

```
OpenAI 가입 → Plus 구독 → Node.js 설치 → Codex CLI 설치
     ↓
codex login → workshop-codex 클론
     ↓
codex 실행 → "hello.js 만들어줘" → node hello.js
```

수고하셨습니다! 🚀
