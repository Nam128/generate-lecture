---
description: "강의 PDF를 한국어 번역 마크다운으로 변환"
argument-hint: "<pdf-filename>"
allowed-tools: ["Bash", "Read", "Write", "Glob"]
---

# 강의 PDF → 한국어 마크다운 변환

입력 인자: $ARGUMENTS
인자 형식: `<pdf-filename>`

예시: `Lecture_4_graph.pdf`

## 인자 파싱

$ARGUMENTS에서:
- `PDF_FILE`: 입력된 PDF 파일명 (예: `Lecture_4_graph.pdf`)
- `BASENAME`: PDF_FILE에서 확장자(`.pdf`)를 제거한 이름 (예: `Lecture_4_graph`)

경로:
- PDF 입력: `algorithm/materials/eng/<PDF_FILE>`
- 이미지 출력: `algorithm/materials/kor/images/`
- 마크다운 출력: `algorithm/materials/kor/<BASENAME>.md`

## Step 1: PDF 메타데이터 확인

```bash
pdfinfo algorithm/materials/eng/<PDF_FILE>
```

총 페이지 수를 확인하고 `TOTAL_PAGES` 변수로 사용한다.

## Step 2: 이미지 변환

### 2-1. PDF → PNG 변환
```bash
pdftoppm -png -r 200 algorithm/materials/eng/<PDF_FILE> /tmp/<BASENAME>_page
```

### 2-2. PNG → WebP 변환
각 PNG 파일을 WebP로 변환한다. 파일명 형식: `<BASENAME>_page_<NN>.webp` (NN은 01부터 시작하는 2자리 숫자)

```bash
for f in /tmp/<BASENAME>_page-*.png; do
  num=$(echo "$f" | grep -oP '\d+(?=\.png)' | tail -1)
  padded=$(printf "%02d" "$num")
  cwebp -q 80 "$f" -o "algorithm/materials/kor/images/<BASENAME>_page_${padded}.webp"
done
```

### 2-3. 임시 PNG 정리
```bash
rm /tmp/<BASENAME>_page-*.png
```

## Step 3: 각 페이지 이미지를 읽고 한국어 번역/설명 작성

1부터 TOTAL_PAGES까지 각 페이지에 대해:

1. Read tool로 `algorithm/materials/kor/images/<BASENAME>_page_<NN>.webp` 이미지를 읽는다
2. 슬라이드 내용을 파악한다
3. 아래 **번역 규칙**에 따라 한국어 번역 + 설명을 작성한다

### 번역 규칙

- **영어 전문 용어 병기:** `한국어(English)` 형태로 병기한다
  - 예: `분할 정복(Divide and Conquer)`, `시간 복잡도(Time Complexity)`
- **슬라이드 제목:** `**볼드**`로 표시한다
- **테이블:** 일반 마크다운 테이블을 사용한다 (blockquote 안에 넣지 않음)
- **코드블록:** 일반 fenced code block (```)을 사용한다
- **Blockquote 사용 금지:** `>` 기호를 사용하지 않는다. 모든 내용은 일반 텍스트로 작성
- 수식은 가능한 한 텍스트로 표현하되, 복잡한 경우 LaTeX (`$...$`)를 사용한다
- 슬라이드의 핵심 내용을 충실히 전달하되, 불필요한 장식적 요소는 생략한다

## Step 4: 마크다운 파일 생성

Write tool로 `algorithm/materials/kor/<BASENAME>.md` 파일을 생성한다.

### 파일 구조

```markdown
# <BASENAME에서 유추한 강의 제목>

**강의명:** Algorithms
**교수:** 서재형 (seojae777@konkuk.ac.kr)
**학기:** 2026 Spring
**소속:** Language Intelligence & Representation Lab (ELION), 건국대학교

---

## Page 1
![Page 1](images/<BASENAME>_page_01.webp)

<한국어 번역 내용>

**Notes:**


---

## Page 2
![Page 2](images/<BASENAME>_page_02.webp)

<한국어 번역 내용>

**Notes:**


---

(... 반복 ...)

## Page <TOTAL_PAGES>
![Page <TOTAL_PAGES>](images/<BASENAME>_page_<NN>.webp)

<한국어 번역 내용>

**Notes:**

```

### 핵심 포맷 규칙

1. 헤더 섹션 뒤의 `---`는 Notes 없이 바로 작성
2. 각 페이지는 `## Page N` + 이미지 + 번역 내용 + `**Notes:**` + 빈 줄 2개 + `---` 순서
3. 마지막 페이지는 `---` 없이 `**Notes:**` + 빈 줄 2개로 끝남
4. 이미지 경로는 상대 경로: `images/<BASENAME>_page_<NN>.webp`

### 참고 템플릿

기존 파일 `algorithm/materials/kor/Lecture_3_sorting.md`를 참고 템플릿으로 사용한다. Glob으로 파일 존재를 확인하고, 필요시 Read로 구조를 확인한다.
