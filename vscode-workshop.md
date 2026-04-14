# VSCode 기본 워크샵

VSCode 설치부터 기본 UI, 파일 변경사항(Source Control) 확인, 스테이징 → 커밋 → 푸시, 추천 확장(Extension)까지 진행합니다.

> Git 설치 / GitHub 계정 / SSH 키 설정은 [[github-workshop]] 를 먼저 완료해 주세요.

---

## 1. VSCode 설치

### macOS

```bash
# Homebrew 사용
brew install --cask visual-studio-code
```

또는 [https://code.visualstudio.com/](https://code.visualstudio.com/) 에서 `.zip` 다운로드.

### Windows

1. [https://code.visualstudio.com/](https://code.visualstudio.com/) 접속
2. `Download for Windows` 클릭 후 설치 파일 실행
3. 설치 옵션에서 **"Add to PATH"**, **"Open with Code"** 체크 권장

### 설치 확인

```bash
code --version
```

> `code` 명령이 안 먹히면 VSCode 실행 → `Cmd/Ctrl + Shift + P` → `Shell Command: Install 'code' command in PATH`.

---

## 2. 기본 UI 살펴보기

```
┌─────────────────────────────────────────────┐
│  [Activity Bar] │   [Editor]                 │
│   📁 Explorer    │                            │
│   🔍 Search      │                            │
│   🔀 Source Ctrl │                            │
│   🐞 Run/Debug   │                            │
│   🧩 Extensions  │                            │
├─────────────────────────────────────────────┤
│  [Panel] Terminal / Problems / Output        │
└─────────────────────────────────────────────┘
```

- **Explorer (📁)** : 폴더/파일 탐색
- **Search (🔍)** : 전체 검색/치환
- **Source Control (🔀)** : Git 변경사항 보기 ← 오늘 핵심
- **Extensions (🧩)** : 플러그인 설치
- **Terminal** : `` Ctrl + ` `` 로 열기

---

## 3. 폴더(프로젝트) 열기

```bash
# 클론한 프로젝트 폴더로 이동 후
cd ~/Desktop/<repo>
code .
```

또는 VSCode에서 `File` → `Open Folder...`

---

## 4. 필수 단축키

| 단축키 (macOS / Windows) | 설명 |
|---|---|
| `Cmd/Ctrl + P` | 파일 빠른 이동 |
| `Cmd/Ctrl + Shift + P` | 커맨드 팔레트 |
| `Cmd/Ctrl + B` | 사이드바 토글 |
| `` Ctrl + ` `` | 터미널 토글 |
| `Cmd/Ctrl + F` | 파일 내 검색 |
| `Cmd/Ctrl + Shift + F` | 전체 검색 |
| `Cmd/Ctrl + /` | 주석 토글 |
| `Option/Alt + ↑/↓` | 줄 이동 |
| `Shift + Option/Alt + ↑/↓` | 줄 복제 |
| `Cmd/Ctrl + D` | 같은 단어 다중 선택 |

---

## 5. Source Control 로 Git 다루기

VSCode 좌측 **🔀 Source Control** 아이콘 (또는 `Ctrl/Cmd + Shift + G`) 클릭.

### 5-1. 변경사항 보기

파일을 수정하면 Source Control 패널에 자동으로 나타납니다.

- **Changes** : 아직 스테이징되지 않은 변경
- **Staged Changes** : 스테이징된 변경 (커밋 대상)

파일명 옆 알파벳 표시:

| 표시 | 의미 |
|---|---|
| `U` | Untracked (새 파일) |
| `M` | Modified (수정됨) |
| `D` | Deleted (삭제됨) |
| `A` | Added (스테이징된 새 파일) |
| `C` | Conflict (충돌) |

### 5-2. diff(변경 내용) 확인

파일명을 **클릭**하면 좌(이전) / 우(현재) 비교 뷰가 열립니다.

- 빨간색: 삭제된 줄
- 초록색: 추가된 줄
- 에디터 좌측의 색 바(gutter)로도 변경 라인을 바로 확인

### 5-3. 스테이징 (Stage)

- 파일 한 개: 파일 오른쪽 `+` 버튼
- 전체: `Changes` 섹션 옆 `+` 버튼 (`git add .` 과 동일)
- **일부 라인만**: diff 뷰에서 변경 블록 좌측의 `···` → `Stage Selected Ranges`
  (→ `git add -p` 와 같은 효과)

### 5-4. 언스테이징 (Unstage)

- `Staged Changes` 의 `-` 버튼 → 다시 Changes 로 돌아감

### 5-5. 커밋 (Commit)

1. 상단 메시지 입력칸에 커밋 메시지 작성
   ```
   feat: 로그인 버튼 추가
   ```
2. `✓ Commit` 버튼 클릭 (또는 `Cmd/Ctrl + Enter`)

> **주의**: 스테이징된 게 하나도 없을 때 커밋하면 "모든 변경을 자동 스테이징할까요?" 라고 물어봄. 의도하지 않았다면 취소.

### 5-6. 푸시 (Push) / 풀 (Pull)

커밋 후:

- **Sync Changes** 버튼 (↻) : pull + push 한 번에
- 혹은 `···` 메뉴 → `Push`, `Pull`

처음 푸시하는 브랜치라면 "Publish Branch" 버튼이 뜹니다. 클릭하면 `git push -u origin <branch>` 와 동일하게 동작.

### 5-7. 브랜치 만들기 / 이동

- 좌측 하단 상태바에 현재 브랜치명 표시 → **클릭**
- `+ Create new branch...` 선택

---

## 6. 실습: 변경 → 스테이지 → 커밋 → 푸시

1. 프로젝트 파일 하나를 수정하고 저장 (`Cmd/Ctrl + S`)
2. Source Control 패널에서 `M` 표시 확인
3. 파일 클릭해 diff 확인
4. `+` 로 스테이징 → `Staged Changes` 로 이동 확인
5. 메시지 입력: `chore: vscode 워크샵 실습`
6. `✓ Commit`
7. `Sync Changes` 로 푸시
8. GitHub 저장소에서 커밋이 올라왔는지 확인 🎉

---

## 7. 추천 확장 (Extensions)

좌측 🧩 아이콘 → 이름 검색 후 `Install`.

### 공통 (거의 모든 프로젝트에 유용)

| 확장 | 설명 |
|---|---|
| **GitLens** | 각 줄에 작성자/커밋 표시(blame), 히스토리 탐색 |
| **GitHub Pull Requests and Issues** | VSCode 안에서 PR/이슈 리뷰 |
| **Prettier - Code formatter** | 저장 시 자동 포맷팅 |
| **EditorConfig for VS Code** | 팀 공용 들여쓰기/개행 규칙 적용 |
| **Error Lens** | 에러/경고를 해당 줄에 인라인 표시 |
| **Code Spell Checker** | 오타 감지 |
| **Todo Tree** | `TODO`, `FIXME` 주석을 한 곳에 모아 보기 |
| **Path Intellisense** | 파일 경로 자동완성 |
| **Material Icon Theme** | 폴더/파일 아이콘을 보기 좋게 |
| **indent-rainbow** | 들여쓰기 색으로 구분 |

### 언어별

**JavaScript / TypeScript / React**
- ESLint
- Prettier
- ES7+ React/Redux/React-Native snippets
- Tailwind CSS IntelliSense (Tailwind 사용 시)

**Python**
- Python (Microsoft)
- Pylance
- Black Formatter / Ruff

**Go**
- Go (Google)

**Docker / K8s / 인프라**
- Docker
- Kubernetes
- HashiCorp Terraform

**마크다운/문서**
- Markdown All in One
- Markdown Preview Enhanced

**원격 / 협업**
- Remote - SSH : 원격 서버에 붙어서 개발
- Dev Containers : 컨테이너 안에서 개발
- Live Share : 실시간 페어 프로그래밍

---

## 8. 추천 설정 (settings.json)

`Cmd/Ctrl + Shift + P` → `Preferences: Open User Settings (JSON)`

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.rulers": [100],
  "editor.minimap.enabled": false,
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "files.eol": "\n",
  "git.autofetch": true,
  "git.confirmSync": false,
  "git.enableSmartCommit": true,
  "workbench.iconTheme": "material-icon-theme",
  "terminal.integrated.fontSize": 13
}
```

- `git.autofetch` : 주기적으로 원격 변경 감지 (푸시 전 pull 놓치기 방지)
- `git.enableSmartCommit` : 스테이징된 게 없으면 전체를 자동 스테이징해 커밋

---

## 9. 문제 해결 팁

- **"code" 명령이 동작하지 않음**
  → 커맨드 팔레트 → `Shell Command: Install 'code' command in PATH`
- **Source Control에 아무것도 안 뜸**
  → 해당 폴더가 Git 저장소가 아님. `git init` 하거나 상위 폴더를 열었는지 확인
- **커밋은 됐는데 GitHub에 안 보임**
  → 푸시를 안 한 것. `Sync Changes` 또는 `Push` 실행
- **Push 시 인증 실패**
  → [[github-workshop]] 의 SSH 키 설정 재확인

---

## 10. 전체 흐름 요약

```
VSCode 설치 → 폴더 열기 → 파일 수정
     ↓
Source Control(🔀) 에서 변경 확인 (diff)
     ↓
+ 스테이징 → 메시지 작성 → ✓ 커밋
     ↓
Sync Changes (= pull + push)
     ↓
GitHub 에서 확인 🎉
```

수고하셨습니다! 🎉
