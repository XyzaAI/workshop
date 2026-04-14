# GitHub 기본 워크샵

Git 설치부터 SSH 설정, 프로젝트 초대, 클론, 커밋, 푸시까지 진행합니다.

---

## 0. 터미널 열기

앞으로 나오는 명령어들은 모두 **터미널(명령어 입력 창)** 에서 실행합니다.

### macOS — 터미널(Terminal)

방법 1) **Spotlight 검색**
- 키보드에서 `Command(⌘) + Space` 누르기
- `터미널` 또는 `Terminal` 입력 후 Enter

방법 2) **Launchpad**
- Launchpad → `기타(Other)` 폴더 → `터미널` 클릭

방법 3) **Finder**
- Finder → `응용 프로그램` → `유틸리티` → `터미널`

> 검은(또는 흰) 창이 열리고 `이름@컴퓨터 ~ %` 같은 문구가 보이면 준비 완료.

### Windows — Git Bash

Git을 설치하면 함께 설치됩니다. (1번 섹션에서 설치 후 사용)

방법 1) **시작 메뉴**
- 키보드 `Windows` 키 누르기
- `Git Bash` 입력 후 Enter

방법 2) **바탕화면/폴더에서**
- 원하는 폴더에서 **빈 공간 우클릭** → `Git Bash Here`
- 해당 폴더 위치에서 바로 터미널이 열립니다.

> `$` 표시가 있는 창이 열리면 준비 완료.

### 터미널 사용 팁 (공통)

| 명령어 | 설명 |
|---|---|
| `pwd` | 현재 내가 있는 폴더 경로 |
| `ls` | 현재 폴더의 파일 목록 |
| `cd 폴더명` | 해당 폴더로 이동 |
| `cd ..` | 상위 폴더로 이동 |
| `clear` | 화면 깨끗이 지우기 |

- 명령어 입력 후 **Enter** 로 실행
- 붙여넣기: macOS `⌘ + V`, Windows Git Bash `Shift + Insert` 또는 우클릭 → Paste

---

## 1. Git 설치

### macOS

```bash
# Homebrew가 없다면 먼저 설치
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Git 설치
brew install git

# 설치 확인
git --version
```

### Windows

1. [https://git-scm.com/download/win](https://git-scm.com/download/win) 접속
2. 설치 파일(Git for Windows) 다운로드 후 실행
3. 옵션은 대부분 기본값으로 두고 `Next` 진행
4. 설치 완료 후 `Git Bash`를 실행하여 확인

```bash
git --version
```

> 이후 모든 명령어는 **Git Bash** (Windows) 또는 **Terminal** (macOS)에서 실행합니다.

---

## 2. Git 기본 설정

이름과 이메일을 등록합니다. (GitHub 계정과 동일한 이메일 권장)

```bash
git config --global user.name "홍길동"
git config --global user.email "you@example.com"

# 기본 브랜치 이름을 main으로
git config --global init.defaultBranch main

# 설정 확인
git config --global --list
```

---

## 3. GitHub 계정 만들기

1. [https://github.com](https://github.com) 접속
2. `Sign up` → 이메일, 비밀번호, 사용자명 입력
3. 이메일 인증 완료

---

## 4. SSH 키 생성 및 GitHub 등록

HTTPS 대신 SSH를 사용하면 매번 비밀번호/토큰을 입력하지 않아도 됩니다.

### 4-1. SSH 키 생성 (macOS / Windows 공통)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

- `Enter file in which to save the key` → 그냥 Enter (기본 경로 사용)
- `passphrase` → 비워 두거나 원하는 암호 입력

### 4-2. 공개키 복사

#### macOS

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

#### Windows (Git Bash)

```bash
cat ~/.ssh/id_ed25519.pub | clip
```

복사가 안 되면 아래로 출력 후 직접 복사:

```bash
cat ~/.ssh/id_ed25519.pub
```

### 4-3. GitHub에 공개키 등록

1. GitHub → 우측 상단 프로필 → `Settings`
2. 좌측 `SSH and GPG keys` → `New SSH key`
3. Title: 원하는 이름 (예: `MacBook`, `Work-PC`)
4. Key: 복사한 공개키 붙여넣기 → `Add SSH key`

### 4-4. 연결 테스트

```bash
ssh -T git@github.com
```

> `Hi <username>! You've successfully authenticated ...` 메시지가 나오면 성공.

---

## 5. 프로젝트(저장소)에 초대받기

저장소 관리자(Owner)가 초대해야 합니다.

### 관리자 입장

1. GitHub 저장소 → `Settings` → `Collaborators`
2. `Add people` → 초대할 사용자 GitHub 아이디/이메일 입력
3. 권한 선택 → `Add`

### 초대받은 입장

1. 등록한 이메일 또는 GitHub 알림에서 초대 메일 확인
2. `View invitation` → `Accept invitation` 클릭

---

## 6. 저장소 클론 (Clone)

초대 수락 후, 저장소의 `Code` 버튼 → `SSH` 탭 → 주소 복사.

```bash
# 작업 폴더로 이동
cd ~/Desktop

# 클론 (이번 워크샵 실습 저장소)
git clone git@github.com:XyzaAI/workshop-git.git

# 폴더로 이동
cd workshop-git
```

---

## 7. 변경사항 작업 → add → commit → push

### 7-1. 파일 수정

원하는 파일을 수정하거나 새 파일을 추가합니다.

```bash
echo "hello" > hello.txt
```

### 7-2. 상태 확인

```bash
git status
```

### 7-3. 스테이징 (add)

```bash
# 특정 파일만
git add hello.txt

# 변경된 파일 전체
git add .
```

### 7-4. 커밋 (commit)

```bash
git commit -m "feat: hello.txt 추가"
```

> 커밋 메시지는 **무엇을 왜 바꿨는지** 짧고 명확하게.

### 7-5. 푸시 (push)

```bash
# 현재 브랜치를 원격으로 올리기
git push origin main
```

처음 푸시하는 새 브랜치라면:

```bash
git push -u origin <branch-name>
```

---

## 8. 자주 쓰는 명령어 정리

| 명령어 | 설명 |
|---|---|
| `git status` | 현재 변경 상태 확인 |
| `git log --oneline` | 커밋 히스토리 보기 |
| `git pull` | 원격 변경사항 받아오기 |
| `git branch` | 브랜치 목록 |
| `git checkout -b <name>` | 새 브랜치 생성 & 이동 |
| `git switch <name>` | 브랜치 이동 |
| `git diff` | 변경 내용 비교 |

---

## 9. 전체 흐름 요약

```
Git 설치 → 사용자 설정 → GitHub 가입 → SSH 키 생성 & 등록
     ↓
저장소 초대 수락 → git clone
     ↓
파일 수정 → git add → git commit → git push
```

수고하셨습니다! 🎉
