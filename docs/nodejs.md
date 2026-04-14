# Node.js 설치 가이드

Node.js는 JavaScript를 터미널에서 실행할 수 있게 해주는 런타임입니다. 함께 설치되는 `npm` 으로 CLI 도구를 설치할 수 있습니다.

> 권장 버전: **Node.js 20 LTS 이상**

---

## 0. 사전 준비

- 터미널 사용 가능 (→ [GitHub 워크샵 0번 섹션](../github-workshop.md) 참고)
- macOS 사용자: [Homebrew 설치](./brew.md) 완료

---

## 1. 설치 여부 먼저 확인

```bash
node -v
npm -v
```

- `v20.x.x` 이상이면 이미 설치 완료 → 바로 원래 워크샵 문서로 돌아가세요.
- `command not found` 라면 아래 설치 진행.

---

## 2. 설치

### macOS

```bash
brew install node
```

### Windows

1. [https://nodejs.org](https://nodejs.org) 접속
2. **LTS** 버전 `.msi` 다운로드
3. 설치 마법사 실행 — 옵션은 전부 기본값으로 `Next`
4. 설치 완료 후 **Git Bash를 종료 후 재시작**

---

## 3. 설치 확인

```bash
node -v
npm -v
```

버전이 출력되면 완료 🎉

---

## 4. Hello World 테스트 (선택)

```bash
node -e "console.log('Hello, Node!')"
```

`Hello, Node!` 가 출력되면 정상 동작.

---

## 5. 자주 쓰는 명령어

| 명령어 | 설명 |
|---|---|
| `node <파일.js>` | JS 파일 실행 |
| `npm install <패키지>` | 현재 폴더에 패키지 설치 |
| `npm install -g <패키지>` | 전역 설치 (CLI 도구용) |
| `npm uninstall -g <패키지>` | 전역 제거 |
| `npm list -g --depth=0` | 전역 설치된 패키지 목록 |

---

## 6. 문제 해결

### `npm install -g` 시 권한 에러
- **macOS**: 앞에 `sudo` 를 붙여 재실행
  ```bash
  sudo npm install -g <패키지>
  ```
- **Windows**: Git Bash를 **관리자 권한으로 실행** 후 재시도

### Windows에서 `node -v` 가 인식되지 않음
- Git Bash(또는 터미널)를 **완전히 종료 후 다시 실행**
- 그래도 안 되면 PC 재부팅

### 버전이 너무 낮음 (v16 이하)
- macOS: `brew upgrade node`
- Windows: [nodejs.org](https://nodejs.org) 에서 LTS 재설치

---

설치가 끝나면 원래 워크샵 문서로 돌아가 다음 단계를 진행하세요.
