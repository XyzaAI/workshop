# Gemini CLI 워크샵

Google 계정 생성부터 요금제 등록, Gemini CLI 설치(mac/windows), 실습 리포지토리 클론, Hello World 실행까지 진행합니다.

> 사전 준비: **GitHub 워크샵**을 먼저 완료하여 Git 설치 / SSH 설정 / 터미널 사용이 가능해야 합니다.

---

## 0. 이 워크샵에서 할 일

1. Google 계정 만들기
2. 요금제(결제수단) 등록
3. Gemini CLI 설치 (macOS / Windows)
4. 로그인
5. 실습 리포지토리 `workshop-gemini-cli` 클론
6. Gemini CLI로 Hello World 만들어보기

---

## 1. Google 계정 만들기

1. [https://accounts.google.com/signup](https://accounts.google.com/signup) 접속
2. 이름 / 사용자 이름 / 비밀번호 입력
3. 전화번호 · 복구 이메일 인증
4. 생년월일 · 성별 입력 후 계정 생성

> 이미 Google 계정이 있다면 그대로 사용하면 됩니다.

---

## 2. 요금제 등록

Gemini CLI는 **Google 계정 로그인 방식**으로 사용할 수 있으며, 무료 티어도 제공됩니다.
실무 레벨에서는 **Gemini Code Assist Standard / Enterprise** 또는 **Google AI Pro/Ultra** 가입을 권장합니다.

### 2-1. 무료로 시작하기

- Google 계정만 있으면 Gemini CLI 무료 티어 사용 가능
- 일/분당 요청 한도가 있으며, 한도 초과 시 제한됨

### 2-2. 유료 플랜 가입 (권장)

1. [https://gemini.google.com](https://gemini.google.com) 로그인
2. 좌측 하단 프로필 → `Upgrade` 또는 [Google One AI Premium](https://one.google.com/about/ai-premium) 접속
3. **Google AI Pro** 이상 선택
4. 결제수단(카드) 등록 → 구독 완료

> API Key 방식으로도 사용 가능합니다. [Google AI Studio](https://aistudio.google.com/apikey) 에서 발급받을 수 있습니다.
> 이 워크샵은 **Google 계정 로그인 방식** 기준으로 진행합니다.

### 2-3. 가입 확인

- 프로필 → `관리` 에서 현재 플랜이 표시되면 OK

---

## 3. Gemini CLI 설치

### 3-1. 사전 요구사항 — Node.js 설치

Gemini CLI는 Node.js 기반입니다. (20 이상 권장)

→ [Node.js 설치 가이드](./docs/nodejs.md) 를 먼저 완료하세요.
(macOS는 Homebrew가 필요합니다 → [Homebrew 설치 가이드](./docs/brew.md))

### 3-2. Gemini CLI 설치 (공통)

터미널(macOS Terminal / Windows Git Bash)에서:

```bash
npm install -g @google/gemini-cli
```

설치 확인:

```bash
gemini --version
```

> 권한 오류가 난다면 macOS는 `sudo npm install -g @google/gemini-cli`, Windows는 Git Bash를 **관리자 권한**으로 실행하세요.

---

## 4. Gemini CLI 로그인

터미널에서 실행:

```bash
gemini
```

- 처음 실행 시 인증 방식을 묻습니다 → `Login with Google` 선택
- 브라우저가 자동으로 열립니다
- Google 계정으로 로그인 → 권한 허용 클릭
- 터미널에 `Authentication successful` 메시지가 뜨면 완료

> API Key 방식을 쓰려면 환경변수로 `export GEMINI_API_KEY="..."` 를 설정하세요.

---

## 5. 실습 리포지토리 클론

`workshop-gemini-cli` 저장소를 받아 실습합니다.

### 5-1. 작업 폴더로 이동

```bash
cd ~/Desktop
```

### 5-2. 클론

```bash
git clone git@github.com:<org>/workshop-gemini-cli.git
cd workshop-gemini-cli
```

> `<org>` 는 강사가 안내하는 조직/계정명으로 바꿔주세요.
> 초대가 필요한 저장소라면 **GitHub 워크샵 5번** 과정대로 초대를 먼저 수락해야 합니다.

### 5-3. 폴더 구조 확인

```bash
ls
```

---

## 6. 실습 — Hello World 만들기

### 6-1. Gemini CLI 실행

저장소 폴더 안에서:

```bash
gemini
```

> 대화형 프롬프트가 열립니다. 여기서 자연어로 Gemini에게 지시할 수 있습니다.

### 6-2. 첫 번째 요청

프롬프트에 다음과 같이 입력:

```
hello.js 파일을 만들어서 "Hello, Gemini!" 를 출력하도록 해줘.
```

Gemini가 파일 생성을 제안하면:
- 변경사항 확인 → `y` (승인) 입력

### 6-3. 결과 실행

Gemini 세션을 잠시 빠져나와(또는 새 터미널에서):

```bash
node hello.js
```

출력:

```
Hello, Gemini!
```

🎉 축하합니다 — 첫 Gemini CLI 실습 성공입니다.

---

## 7. 한 걸음 더 — 연습 과제

Gemini 프롬프트에 아래를 순서대로 시도해 보세요.

1. `hello.js` 가 이름을 인자로 받아 `Hello, <이름>!` 을 출력하도록 바꿔줘.
2. 같은 기능을 하는 `hello.py` 도 만들어줘.
3. `README.md` 에 실행 방법을 적어줘.

각 단계마다 결과를 실행해 확인:

```bash
node hello.js Kang
python3 hello.py Kang
```

---

## 8. 자주 쓰는 Gemini CLI 명령어

| 명령어 | 설명 |
|---|---|
| `gemini` | 대화형 세션 시작 |
| `gemini -p "프롬프트"` | 한 번만 실행 (non-interactive) |
| `gemini --help` | 전체 옵션 보기 |
| `gemini --version` | 버전 확인 |

세션 안에서:
- `/help` — 내부 명령어 보기
- `/auth` — 인증 방식 변경
- `/chat` — 대화 저장/복원
- `/tools` — 사용 가능한 도구 목록
- `/quit` 또는 `Ctrl + C` — 세션 종료

---

## 9. 전체 흐름 요약

```
Google 가입 → (선택) AI Pro 구독 → Node.js 설치 → Gemini CLI 설치
     ↓
gemini 실행 → Google 로그인 → workshop-gemini-cli 클론
     ↓
gemini 실행 → "hello.js 만들어줘" → node hello.js
```

수고하셨습니다! 🚀
