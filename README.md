# generate-lecture

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash command that converts lecture PDF slides into structured Korean-translated Markdown — optimized for [Obsidian](https://obsidian.md/) note-taking.

## What it does

```
/generate-lecture Lecture_4_graph.pdf 4 graph
```

This single command triggers Claude to:

1. **Extract** page count from the PDF (`pdfinfo`)
2. **Convert** each page to optimized WebP images (`pdftoppm` → `cwebp`)
3. **Read** each slide image using Claude's vision capability
4. **Translate** slide content into Korean with English terminology preserved
5. **Generate** a complete Markdown file with per-page notes sections

### Output format

```markdown
## Page 1
![Page 1](images/lec4_page_01.webp)

**그래프 탐색(Graph Traversal)**

그래프(Graph)에서 모든 정점(Vertex)을 방문하는 알고리즘...

**Notes:**


---
```

Each page includes:
- Original slide image (WebP)
- Korean translation with `한국어(English)` bilingual terms
- `**Notes:**` section for your own study notes
- Clean Markdown — no blockquotes, proper tables and code blocks

## Installation

### Option 1: Copy the file (simplest)

```bash
# In your project root
mkdir -p .claude/commands
curl -o .claude/commands/generate-lecture.md \
  https://raw.githubusercontent.com/Nam128/generate-lecture/main/generate-lecture.md
```

### Option 2: Clone and symlink

```bash
git clone https://github.com/Nam128/generate-lecture.git ~/.claude-skills/generate-lecture
ln -s ~/.claude-skills/generate-lecture/generate-lecture.md \
  /path/to/your/project/.claude/commands/generate-lecture.md
```

## Prerequisites

The following CLI tools must be installed on your system:

| Tool | Purpose | Install (macOS) | Install (Ubuntu) |
|------|---------|-----------------|------------------|
| `pdfinfo` | Read PDF metadata | `brew install poppler` | `apt install poppler-utils` |
| `pdftoppm` | PDF → PNG conversion | (included with poppler) | (included with poppler-utils) |
| `cwebp` | PNG → WebP conversion | `brew install webp` | `apt install webp` |

**macOS (one-liner):**
```bash
brew install poppler webp
```

**Ubuntu/Debian:**
```bash
sudo apt install poppler-utils webp
```

## Usage

### Expected directory structure

```
your-project/
├── .claude/
│   └── commands/
│       └── generate-lecture.md    ← this skill
├── algorithm/
│   └── materials/
│       ├── eng/                   ← put your PDF here
│       │   └── Lecture_4_graph.pdf
│       └── kor/
│           ├── images/            ← WebP images go here (auto-created)
│           └── Lecture_4_graph.md  ← output Markdown
```

### Run the command

```bash
claude  # start Claude Code
```

Then type:
```
/generate-lecture Lecture_4_graph.pdf 4 graph
```

**Arguments:**

| Position | Name | Example | Description |
|----------|------|---------|-------------|
| 1 | `pdf-filename` | `Lecture_4_graph.pdf` | PDF file in `algorithm/materials/eng/` |
| 2 | `lecture-num` | `4` | Lecture number (used for image naming) |
| 3 | `topic` | `graph` | Topic slug (used for output filename) |

## Translation rules

The skill instructs Claude to follow these rules when translating:

- **Bilingual terms:** `한국어(English)` format — e.g., `시간 복잡도(Time Complexity)`
- **Slide titles:** Rendered in **bold**
- **Tables:** Standard Markdown tables (no blockquotes)
- **Code blocks:** Fenced code blocks with language hints
- **No blockquotes:** All content as plain text for clean Obsidian rendering
- **Math:** Text when simple, LaTeX (`$...$`) when complex

## Customization

The skill is a plain Markdown file — edit it freely to fit your needs:

- **Change language:** Replace Korean translation rules with your target language
- **Change paths:** Modify `algorithm/materials/eng/` and `kor/` to match your structure
- **Change header:** Update professor name, university, semester info
- **Add fields:** Add custom metadata to the page template
- **Adjust image quality:** Change `cwebp -q 80` to your preferred quality level

## Example output

See [Lecture_3_sorting.md](https://github.com/Nam128/study-log/blob/main/algorithm/materials/kor/Lecture_3_sorting.md) for a complete example of the generated output format (56 pages, Korean-translated with notes sections).

## License

MIT
