# Obsidian 기본 워크샵

Obsidian 설치부터 Vault 생성, 기본 문법, 링크/백링크, 폴더 구조, 플러그인, Git 연동까지 진행합니다.

---

## 1. Obsidian이란?

- 로컬 마크다운(`.md`) 파일 기반의 개인 지식 관리(PKM) 툴
- 모든 노트는 내 컴퓨터에 **평문 마크다운**으로 저장됨 (락인 없음)
- 노트 간 **양방향 링크(백링크)** 와 **그래프 뷰** 지원
- 플러그인 생태계가 강력함 (코어 + 커뮤니티)

---

## 2. 설치

### macOS

```bash
# Homebrew 사용
brew install --cask obsidian
```

또는 [https://obsidian.md/download](https://obsidian.md/download) 에서 `.dmg` 다운로드.

### Windows

1. [https://obsidian.md/download](https://obsidian.md/download) 접속
2. Windows Installer 다운로드 후 실행
3. 기본값으로 설치

---

## 3. Vault(저장소) 만들기

Obsidian에서 "Vault"는 노트들이 모여있는 **하나의 폴더**입니다.

1. Obsidian 실행 → `Create new vault`
2. Vault 이름: 예) `my-notes`
3. 위치: 원하는 폴더 (예: `~/Desktop/my-notes`)
4. `Create`

> 이미 있는 폴더를 열려면 `Open folder as vault` 선택.

---

## 4. 첫 노트 작성

좌측 사이드바에서 `New note` → 파일명 입력 후 본문 작성.

```markdown
# 오늘의 메모

- Obsidian 설치 완료
- [[github-workshop]] 도 같이 읽어보기
```

---

## 5. 마크다운 기본 문법

```markdown
# 제목 1
## 제목 2
### 제목 3

**굵게**, *기울임*, ~~취소선~~, `인라인 코드`

- 리스트 1
- 리스트 2
  - 중첩

1. 순서 있는 리스트
2. 두 번째

> 인용문

- [ ] 할 일
- [x] 완료한 일

[링크 텍스트](https://obsidian.md)

![이미지 설명](image.png)

\`\`\`bash
echo "코드 블록"
\`\`\`

| 헤더1 | 헤더2 |
|---|---|
| 값1 | 값2 |
```

---

## 6. Obsidian만의 핵심 기능

### 6-1. 내부 링크 `[[ ]]`

```markdown
[[github-workshop]]           # 같은 Vault의 노트로 이동
[[github-workshop|깃허브 워크샵]]  # 표시 텍스트 변경
[[github-workshop#3장]]       # 특정 헤딩으로 점프
```

- 존재하지 않는 노트명을 링크하면 **클릭 시 자동 생성**됨

### 6-2. 백링크 (Backlinks)

- 우측 패널 → `Backlinks`
- 이 노트를 **링크하고 있는 다른 노트들**이 자동으로 보임

### 6-3. 태그

```markdown
#workshop #obsidian/basic
```

좌측 `Tags` 패널에서 태그별로 모아볼 수 있음.

### 6-4. 그래프 뷰 (Graph View)

- 좌측 사이드바의 그래프 아이콘 클릭
- 노트 간 연결 관계를 시각적으로 탐색

### 6-5. 커맨드 팔레트

- `Cmd/Ctrl + P` → 모든 명령 검색 실행
- `Cmd/Ctrl + O` → 노트 빠른 이동

---

## 7. 추천 폴더 구조 (예시)

정답은 없습니다. 처음엔 단순하게 시작하세요.

```
my-notes/
├── 00-inbox/        # 빠르게 적고 나중에 정리
├── 10-daily/        # 데일리 노트
├── 20-projects/     # 진행 중인 프로젝트
├── 30-areas/        # 꾸준히 관리하는 영역 (업무/학습)
├── 40-resources/    # 참고자료
└── 90-archive/      # 종료된 것들
```

---

## 8. 유용한 코어 플러그인 (기본 내장)

`Settings` → `Core plugins` 에서 활성화.

| 플러그인 | 설명 |
|---|---|
| Daily notes | 날짜별 노트 자동 생성 |
| Templates | 템플릿 삽입 |
| Backlinks | 역참조 표시 |
| Outgoing links | 이 노트가 가리키는 링크 목록 |
| Tag pane | 태그 패널 |
| Command palette | 명령 팔레트 |
| Quick switcher | 빠른 노트 이동 |

---

## 9. 추천 커뮤니티 플러그인

`Settings` → `Community plugins` → `Turn on community plugins` → `Browse`.

- **Dataview** : 노트를 DB처럼 쿼리
- **Templater** : 강력한 템플릿 엔진
- **Calendar** : 달력 UI로 데일리 노트 관리
- **Excalidraw** : 손그림/다이어그램
- **Git** : Vault를 Git으로 자동 동기화

---

## 10. Git으로 Vault 백업 / 동기화

Vault는 결국 마크다운 폴더이므로 Git과 궁합이 좋습니다.
(GitHub 설정은 [[github-workshop]] 참고)

```bash
cd ~/Desktop/my-notes

git init
git add .
git commit -m "chore: init vault"

# GitHub에 빈 저장소를 만든 뒤
git remote add origin git@github.com:<owner>/my-notes.git
git branch -M main
git push -u origin main
```

이후 커뮤니티 플러그인 **Obsidian Git** 을 설치하면 주기적으로 자동 커밋/푸시가 가능합니다.

---

## 11. 자주 쓰는 단축키

| 단축키 (macOS / Windows) | 설명 |
|---|---|
| `Cmd/Ctrl + N` | 새 노트 |
| `Cmd/Ctrl + O` | 빠른 노트 이동 |
| `Cmd/Ctrl + P` | 커맨드 팔레트 |
| `Cmd/Ctrl + E` | 편집/읽기 모드 전환 |
| `Cmd/Ctrl + ,` | 설정 |
| `Cmd/Ctrl + 클릭` | 링크 따라가기 |
| `Alt/Option + ←` | 뒤로 가기 |

---

## 12. 실습 과제

1. 새 Vault `my-notes` 생성
2. `hello.md` 노트 만들고 마크다운 문법으로 자기소개 작성
3. `obsidian-workshop.md` 를 `[[ ]]` 로 링크해 보기
4. 태그 `#workshop` 달기
5. 그래프 뷰에서 연결이 보이는지 확인
6. (선택) Vault를 GitHub에 푸시

---

## 13. 전체 흐름 요약

```
Obsidian 설치 → Vault 생성 → 마크다운으로 노트 작성
     ↓
[[내부 링크]] + #태그 로 연결
     ↓
백링크 / 그래프 뷰로 지식 탐색
     ↓
플러그인으로 확장 → Git으로 백업
```

수고하셨습니다! 🎉
