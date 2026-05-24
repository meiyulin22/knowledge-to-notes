---
name: knowledge-to-notes
description: "Transform messy notes, PDFs, and documents into beautifully formatted, Notion-ready markdown notes with rich emoji, clear hierarchy, and UX-driven layout. Use when the user wants to clean up study notes, convert documents into readable knowledge cards, or organize text into visually appealing markdown."
compatibility: "Claude Code skill directories (~/.claude/skills) and Amp skill directories (~/.config/agents/skills, ~/.config/amp/skills)"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
argument-hint: <path-to-document> [output-name]
---

# Knowledge to Notes (k2n)

Transform raw documents into beautifully formatted, human-readable study notes optimized for Notion, Obsidian, or any markdown editor.

## Philosophy

Most notes are ugly. Dense walls of text crush motivation before reading begins. This skill treats note formatting as **UX design** — every heading, emoji, table, and callout serves a purpose: make the reader *want* to learn.

**Format for humans, not machines.** A beautiful note uses visual hierarchy, strategic whitespace, emoji landmarks, and bite-sized chunks to create flow state. The goal is notes so pleasant to read that users lose track of time.

## What It Does

1. Extract text from any supported document (PDF, MD, TXT, EPUB, DOCX, HTML, RTF, MOBI/AZW)
2. Analyze structure and identify key concepts, hierarchies, and patterns
3. Format as a stunning markdown note with:
   - Rich emoji section markers for instant visual navigation
   - Multi-level tables for comparison and reference
   - Blockquote insights (💡), warnings (⚠️), and traps (❌→✅)
   - Code blocks with language labels where applicable
   - Strategic horizontal rules separating major sections
   - Memory hooks and mnemonics where the content allows

## Usage

```
/knowledge-to-notes <path-to-document> [output-name]
```

**Examples:**
```
/knowledge-to-notes ~/Desktop/grammar.pdf english-grammar
/knowledge-to-notes ~/notes/react-notes.md
/knowledge-to-notes ~/books/clean-code.pdf
```

**Output:** `~/Desktop/<name>-知识笔记.md` — open, copy, paste into Notion/Obsidian.

---

## Step 1 — Validate Input

```bash
test -f "$DOC_PATH" && echo "FILE_OK" || echo "FILE_NOT_FOUND"
```

If not found or unsupported format, report error and list supported formats: PDF, EPUB, DOCX, TXT, MD, HTML, RTF, MOBI/AZW.

---

## Step 2 — Identify Content Type

Ask the user (or auto-detect from filename/extension):

> "What kind of content is this?
> 1. **Technical** — code, formulas, tables (programming, math, science)
> 2. **Conceptual** — frameworks, definitions, prose (humanities, business, theory)
> 3. **Mixed** — both technical and conceptual
> 4. **Not sure** — I'll auto-detect"

Store as `CONTENT_TYPE`: `technical`, `conceptual`, `mixed`.

---

## Step 3 — Extract Text

Run the extraction script. The script auto-detects format and picks the best extractor:

```bash
# Find extract.py - check skill locations in priority order
SCRIPT_PATH=""
for candidate in \
  ".agents/skills/knowledge-to-notes/scripts/extract.py" \
  "$HOME/.config/agents/skills/knowledge-to-notes/scripts/extract.py" \
  "$HOME/.config/amp/skills/knowledge-to-notes/scripts/extract.py" \
  "$HOME/.claude/skills/knowledge-to-notes/scripts/extract.py"
do
  if [ -f "$candidate" ]; then
    SCRIPT_PATH="$candidate"
    break
  fi
done

# Find Python
PYTHON_BIN="${PYTHON_BIN:-python3}"
if ! command -v "$PYTHON_BIN" >/dev/null 2>&1; then
  PYTHON_BIN="python"
fi

# Run extraction — AI decides mode based on file type
# For PDFs that may be image-based → use paddleocr
# For text PDFs → use text
# For MD/TXT → direct read (no mode needed)
"$PYTHON_BIN" "$SCRIPT_PATH" "$DOC_PATH" --mode $EXTRACTION_MODE --install-missing yes
```

This produces:
- `<tempdir>/k2n_work/full_text.txt` — extracted text
- `<tempdir>/k2n_work/metadata.json` — stats

Read the extracted text.

---

## Step 4 — Analyze Structure

Read the full extracted text and identify:
- **Main topic** and subtopics
- **Natural section breaks** (headings, chapter markers, topic shifts)
- **Key entities**: terms, definitions, formulas, people, dates
- **Patterns**: comparisons, sequences, hierarchies, cause-effect
- **Difficulty spikes**: complex sections that need extra clarity

---

## Step 5 — Format with UX Principles

Generate the output note. Apply these rules:

### Visual Hierarchy
- `#` H1 = document title (one only, with 📗 or 📘 prefix)
- `##` H2 = major sections (with emoji landmarks)
- `###` H3 = sub-sections
- `####` H4 = detail points (use sparingly)

### Emoji Landmarks (pick relevant ones, don't overuse)
```
📗 📘 📙 📚 → Document/book types
🎯 → Learning objectives, key goals
📊 → Tables, comparisons, data
💡 → Key insight, "aha!" moment
⚠️ → Common mistake, gotcha
❌ → Wrong way / ✅ → Right way
🔑 → Key term, definition
💎 → Summary, TL;DR, core takeaway
🔄 → Process, cycle, workflow
📝 → Practice, exercise, action item
🧠 → Memory hook, mnemonic
⏱  → Timeline, history, sequence
```

### Content Blocks (use liberally)
```markdown
> 💡 **Key Insight:** <one-sentence revelation>

> ⚠️ **Common Trap:** <what people get wrong>
> ❌ <wrong approach>
> ✅ <right approach>

> 💎 **TL;DR:** <one paragraph that captures everything>

> 🧠 **Memory Hook:** <mnemonic or vivid association>

> 📝 **Action:** <something to try or practice>
```

### Tables
- Comparison tables: use markdown tables with emoji in headers
- Multi-column data: align carefully
- Keep tables under 6 columns for readability

### Code Blocks (technical content only)
- Always specify language: ```python, ```javascript, ```bash
- Add a one-line comment at top explaining what the code demonstrates
- Keep snippets under 20 lines

### Section Endings
After each major section, add one of:
- `> 💎 **Section Summary:** ...`
- A mini-table of key points
- A transitional sentence to the next section

---

## Step 6 — Write Output

Write to the user's Desktop:

```
~/Desktop/<output-name>-知识笔记.md
```

If `output-name` not provided, derive from filename (strip extension, replace underscores/hyphens with spaces).

---

## Step 7 — Report

```
✅ Note created: ~/Desktop/<name>-知识笔记.md

📄 Source: <filename> (<format>)
📊 Sections: <N> | Tables: <N> | Insights: <N>

Copy the file content and paste into Notion, Obsidian, or any markdown editor.
```

---

## Quality Rules

1. **One glance, one answer** — each section should answer one question immediately
2. **Emoji with purpose** — every emoji marks a semantic category, never decoration
3. **Table or text, never both** — don't repeat table content in prose
4. **Bite-sized chunks** — no paragraph over 4 lines
5. **Front-load value** — most important insight in the first section
6. **End with mastery** — final section should make the reader feel they've mastered the topic
7. **No raw walls of text** — break up anything longer than 6 lines
8. **Language matches source** — output in the same language as the input document
