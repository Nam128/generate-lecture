# lecture-pdf-to-korean-md

강의 PDF 슬라이드를 한국어 번역 마크다운으로 변환하는 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash command입니다. [Obsidian](https://obsidian.md/) 노트 테이킹에 최적화되어 있습니다.

## 기능

```
/generate-lecture Lecture_4_graph.pdf
                  ────────────────────
                  PDF 파일명 → BASENAME: Lecture_4_graph
                               ├─ 마크다운: Lecture_4_graph.md
                               └─ 이미지:   Lecture_4_graph_page_01.webp
```

PDF 파일명 하나만 입력하면 확장자를 제거한 `BASENAME`을 자동 추출하여 모든 명명에 사용합니다:
- `Lecture_4_graph.pdf` → BASENAME = `Lecture_4_graph`
- 출력 마크다운: `Lecture_4_graph.md`
- 이미지 파일명: `Lecture_4_graph_page_01.webp`, `Lecture_4_graph_page_02.webp`, ...

이 명령어 하나로 Claude가 다음을 수행합니다:

1. **PDF 분석** — 페이지 수 추출 (`pdfinfo`)
2. **이미지 변환** — 각 페이지를 최적화된 WebP 이미지로 변환 (`pdftoppm` → `cwebp`)
3. **슬라이드 읽기** — Claude의 비전(vision) 기능으로 각 슬라이드 이미지를 읽음
4. **한국어 번역** — 영어 전문 용어를 병기하며 한국어로 번역
5. **마크다운 생성** — 페이지별 노트 섹션이 포함된 마크다운 파일 생성

### 출력 형식

```markdown
## Page 1
![Page 1](images/Lecture_4_graph_page_01.webp)

**그래프 탐색(Graph Traversal)**

그래프(Graph)에서 모든 정점(Vertex)을 방문하는 알고리즘...

**Notes:**


---
```

각 페이지에 포함되는 내용:
- 원본 슬라이드 이미지 (WebP)
- `한국어(English)` 형태의 이중 언어 번역
- `**Notes:**` 섹션 — 직접 메모를 작성할 수 있는 공간
- 깔끔한 마크다운 — blockquote 없이 테이블, 코드블록 등 표준 서식 사용

## 설치

### 방법 1: 파일 복사 (가장 간단)

```bash
# 프로젝트 루트에서 실행
mkdir -p .claude/commands
curl -o .claude/commands/generate-lecture.md \
  https://raw.githubusercontent.com/Nam128/lecture-pdf-to-korean-md/main/generate-lecture.md
```

### 방법 2: Clone 후 심볼릭 링크

```bash
git clone https://github.com/Nam128/lecture-pdf-to-korean-md.git ~/.claude-skills/lecture-pdf-to-korean-md
ln -s ~/.claude-skills/lecture-pdf-to-korean-md/generate-lecture.md \
  /path/to/your/project/.claude/commands/generate-lecture.md
```

## 사전 요구사항

다음 CLI 도구가 시스템에 설치되어 있어야 합니다:

| 도구 | 용도 | macOS 설치 | Ubuntu 설치 |
|------|------|------------|-------------|
| `pdfinfo` | PDF 메타데이터 읽기 | `brew install poppler` | `apt install poppler-utils` |
| `pdftoppm` | PDF → PNG 변환 | (poppler에 포함) | (poppler-utils에 포함) |
| `cwebp` | PNG → WebP 변환 | `brew install webp` | `apt install webp` |

**macOS (한 줄 설치):**
```bash
brew install poppler webp
```

**Ubuntu/Debian:**
```bash
sudo apt install poppler-utils webp
```

## 사용법

### 디렉토리 구조

```
your-project/
├── .claude/
│   └── commands/
│       └── generate-lecture.md    ← 이 skill 파일
├── algorithm/
│   └── materials/
│       ├── eng/                   ← PDF 파일 위치
│       │   └── Lecture_4_graph.pdf
│       └── kor/
│           ├── images/            ← WebP 이미지 (자동 생성)
│           └── Lecture_4_graph.md  ← 출력 마크다운
```

### 명령어 실행

```bash
claude  # Claude Code 실행
```

그다음 입력:
```
/generate-lecture Lecture_4_graph.pdf
```

**인자 설명:**

| 이름 | 예시 | 설명 |
|------|------|------|
| `pdf-filename` | `Lecture_4_graph.pdf` | `algorithm/materials/eng/` 안의 PDF 파일명. 확장자를 제거한 BASENAME이 출력 파일명과 이미지 파일명에 사용됨 |

## 번역 규칙

이 skill은 Claude에게 다음 번역 규칙을 지시합니다:

- **용어 병기:** `한국어(English)` 형태 — 예: `시간 복잡도(Time Complexity)`
- **슬라이드 제목:** **볼드**로 표시
- **테이블:** 표준 마크다운 테이블 (blockquote 사용 안 함)
- **코드블록:** 언어 힌트가 포함된 fenced code block
- **Blockquote 금지:** Obsidian에서 깔끔하게 렌더링되도록 모든 내용은 일반 텍스트로 작성
- **수식:** 간단한 경우 텍스트, 복잡한 경우 LaTeX (`$...$`)

## 커스터마이징

skill 파일은 일반 마크다운이므로 자유롭게 수정할 수 있습니다:

- **언어 변경:** 한국어 번역 규칙을 원하는 언어로 교체
- **경로 변경:** `algorithm/materials/eng/`, `kor/` 등을 프로젝트 구조에 맞게 수정
- **헤더 변경:** 교수명, 대학교, 학기 정보 업데이트
- **필드 추가:** 페이지 템플릿에 원하는 메타데이터 추가
- **이미지 품질 조정:** `cwebp -q 80`의 품질 값을 원하는 수준으로 변경

## 출력 예시

생성된 출력 형식의 전체 예시는 [Lecture_3_sorting.md](https://github.com/Nam128/study-log/blob/main/algorithm/materials/kor/Lecture_3_sorting.md)를 참고하세요 (56페이지, 한국어 번역 + 노트 섹션 포함).

## 라이선스

MIT
