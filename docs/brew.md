# Homebrew 설치 가이드 (macOS)

Homebrew는 macOS에서 개발 도구를 터미널 명령어로 쉽게 설치·관리할 수 있게 해주는 **패키지 매니저**입니다.

> Windows 사용자는 Homebrew를 설치하지 않아도 됩니다. 대신 각 프로그램의 공식 설치 파일(exe) 또는 `winget` 을 사용합니다.

---

## 0. 사전 준비

- macOS 사용자
- 터미널 사용 가능 (→ [GitHub 워크샵 0번 섹션](../github-workshop.md) 참고)
- 인터넷 연결

---

## 1. 설치 여부 먼저 확인

이미 설치되어 있을 수 있으니 먼저 확인합니다.

```bash
brew --version
```

- `Homebrew 4.x.x` 같이 나오면 **이미 설치됨** → 2번(업데이트)로 이동
- `command not found: brew` 라면 아래 설치 진행

---

## 2. Homebrew 설치

터미널에 아래 한 줄을 복사해 붙여넣고 Enter:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- 중간에 macOS 비밀번호 입력 요청 → 로그인 비밀번호 입력 (화면에 표시되지 않아도 정상)
- `Press RETURN to continue` 가 나오면 Enter
- 완료까지 수 분 소요됩니다.

---

## 3. PATH 설정 (Apple Silicon: M1/M2/M3/M4 맥)

Apple Silicon 맥은 설치 후 `brew` 명령을 인식하도록 PATH 설정이 필요합니다.
설치 완료 메시지 맨 아래 `Next steps:` 에 안내되는 명령 2줄을 그대로 실행하세요. 예시:

```bash
echo >> ~/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

> Intel 맥은 이 단계가 필요 없습니다.

---

## 4. 설치 확인

```bash
brew --version
brew doctor
```

- `Your system is ready to brew.` 가 나오면 완료 🎉

---

## 5. 자주 쓰는 Homebrew 명령어

| 명령어 | 설명 |
|---|---|
| `brew install <이름>` | CLI 도구 설치 (예: `brew install git`) |
| `brew install --cask <이름>` | GUI 앱 설치 (예: `brew install --cask obsidian`) |
| `brew uninstall <이름>` | 제거 |
| `brew list` | 설치된 목록 |
| `brew update` | Homebrew 자체 업데이트 |
| `brew upgrade` | 설치된 패키지 전체 업데이트 |
| `brew search <이름>` | 패키지 검색 |

---

## 6. 문제 해결

### `command not found: brew`
- 3번 PATH 설정을 하지 않은 경우. 해당 명령 실행 후 터미널 재시작.

### 설치 중 `xcode-select` 관련 에러
```bash
xcode-select --install
```
Command Line Tools 설치 창이 뜨면 `설치` 클릭.

### 권한 에러
- `sudo` 를 붙이지 말고, 에러 메시지에 적힌 `chown` 명령을 그대로 실행.

---

설치가 끝나면 원래 워크샵 문서로 돌아가 다음 단계를 진행하세요.
