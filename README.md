# Knowledge to Notes (k2n)

> **Transform messy documents into stunning, Notion-ready study notes.**

Tired of dense PDF walls of text that kill your motivation before you even start reading? Knowledge to Notes turns raw documents — PDFs, Markdown files, EPUBs, Word docs — into beautifully formatted markdown notes with rich visual hierarchy, strategic emoji landmarks, and bite-sized chunks that make learning addictive.

## What Problem This Solves

| Before | After |
|--------|-------|
| Ugly, dense walls of text | Clean, scannable sections with emoji landmarks |
| No visual hierarchy | Multi-level headings, tables, and callouts |
| Hard to find key points | 💡 Insights, ⚠️ Traps, 💎 TL;DRs highlighted |
| One giant blob | Bite-sized chunks (no paragraph over 4 lines) |
| "I'll read this later" (never) | "I want to read this NOW" |

## What It Does

1. **Extracts** text from PDF (with OCR for image-based PDFs), EPUB, DOCX, Markdown, HTML, TXT, RTF, MOBI/AZW
2. **Analyzes** structure — identifies topics, subtopics, key concepts, tables, code blocks
3. **Formats** with UX design principles — emoji navigation, comparison tables, insight callouts, memory hooks
4. **Outputs** a single clean `.md` file — copy and paste into Notion, Obsidian, or any markdown editor

## Demo

```
Input:  A messy React study note (Markdown)

Output: Desktop/react-hooks-知识笔记.md
```

### Before (raw notes):
```
React Hooks
useState lets you add state to functional components.
useEffect runs after render. Cleanup with return function.
useContext avoids prop drilling. useReducer for complex state.
useMemo caches values. useCallback caches functions.
Custom hooks let you reuse stateful logic between components.
```

### After (formatted note):
```markdown
📗  React Hooks — The Complete Cheatsheet

> 💡 One sentence: Hooks let you "hook into" React state and lifecycle
  from plain functions — no classes needed.

🎯 The 7 Essential Hooks

📊 Hook Overview
| Hook | Purpose | Returns | When to use |
|------|---------|---------|-------------|
| 🪝 useState | Local state | [value, setter] | Any component state |
| 🔄 useEffect | Side effects | cleanup fn | API calls, timers, DOM |
| 🌐 useContext | Global state | context value | Avoid prop drilling |
| 🗃️ useReducer | Complex state | [state, dispatch] | Multi-step state logic |
| 🧠 useMemo | Cache value | memoized value | Expensive calculations |
| ⚡ useCallback | Cache function | memoized fn | Stable callbacks to children |
| 🛠️ Custom Hooks | Reuse logic | your choice | Shared behavior |

💎 Key Insight
> useState + useEffect cover 80% of hooks usage.
  Master those two before the others.

⚡ Common Trap
❌ Calling setState and reading state immediately (stale closure)
✅ Use the functional updater: setCount(prev => prev + 1)

📝 Practice: Build a useWindowSize custom hook in 5 minutes
```

## Quick Start

### Prerequisites

- [Claude Code](https://claude.ai/code) or [Amp](https://ampcode.com/)
- Python 3.9+

### Installation

```bash
# Clone into Claude Code skills directory
git clone https://github.com/meiyulin22/knowledge-to-notes.git ~/.claude/skills/knowledge-to-notes

# Install Python dependencies (for document extraction)
pip install "paddleocr[all]" paddlepaddle==3.2.1

# Or install only what you need:
# For text PDFs only:     pip install PyPDF2 pdfminer.six
# For image-based PDFs:   pip install "paddleocr[all]" paddlepaddle==3.2.1
# For EPUB:               pip install ebooklib beautifulsoup4
# For DOCX:               pip install python-docx

# ⚠️ Windows users: paddlepaddle 3.3.1 has a OneDNN bug.
# Use 3.2.1:  pip install paddlepaddle==3.2.1
```

### Usage

In Claude Code:

```
/knowledge-to-notes <path-to-document> [output-name]
```

**Examples:**
```
/knowledge-to-notes ~/Desktop/my-notes.md
/knowledge-to-notes ~/books/clean-code.pdf clean-code
/knowledge-to-notes ~/Downloads/article.pdf
```

**Output:** `~/Desktop/<name>-知识笔记.md`

Open the file, copy all content, paste into Notion or Obsidian. Done.

## Supported Formats

| Format | Extraction Method | Notes |
|--------|-------------------|-------|
| **PDF** (text) | pdftotext / PyPDF2 / pdfminer | Auto-detected |
| **PDF** (image-only) | PaddleOCR PP-StructureV3 | Requires `paddleocr[all]` + `paddlepaddle` |
| **Markdown / TXT** | Direct read | Instant |
| **EPUB** | ebooklib + BeautifulSoup4 | Best quality |
| **DOCX** | python-docx | Tables preserved |
| **HTML** | BeautifulSoup4 | Clean text extraction |
| **RTF** | striprtf | - |
| **MOBI / AZW** | Calibre ebook-convert | Requires Calibre installed |

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────────┐
│  PDF/MD/    │────▶│  extract.py  │────▶│  Claude Code  │────▶│  Beautiful │
│  EPUB/DOCX  │     │  text/OCR    │     │  Analyze +    │     │  .md note  │
│             │     │  extraction  │     │  Format       │     │  on Desktop │
└─────────────┘     └──────────────┘     └───────────────┘     └────────────┘
     Input           Python (free)        AI (token cost)        Output
```

- **extract.py**: Python script that auto-detects format and picks the best extraction method. For image-based PDFs, it uses PaddleOCR's PP-StructureV3 pipeline (12 AI models for layout analysis, text detection/recognition, table extraction, and formula recognition). All OCR runs locally — no API calls.
- **Claude Code**: Reads extracted text, analyzes structure, and formats it with UX-driven design principles into a polished markdown note.

## Design Philosophy

> Format for humans, not machines.

Every design choice serves one goal: make the reader *want* to keep reading.

- **Emoji landmarks** create instant visual navigation (not decoration)
- **Bite-sized chunks** prevent cognitive overload (no paragraph over 4 lines)
- **Front-loaded insights** give immediate value (most important point first)
- **TL;DR sections** let you grasp a topic in 10 seconds
- **Mistake callouts** (❌→✅) make learning stick

## Tech Stack

| Layer | Technology |
|-------|-----------|
| AI formatting | Claude Code / Amp (LLM agent) |
| OCR engine | PaddleOCR PP-StructureV3 (12-model pipeline) |
| PDF extraction | pdftotext (poppler), PyPDF2, pdfminer.six |
| EPUB extraction | ebooklib + BeautifulSoup4 |
| DOCX extraction | python-docx |
| Layout analysis | PP-DocLayout, PP-DocBlockLayout (deep learning models) |
| Text recognition | PP-OCRv5 (109-language OCR) |
| Table recognition | SLANeXt, SLANet+, RT-DETR-L |
| Formula recognition | PP-FormulaNet |

## Project Structure

```
knowledge-to-notes/
├── SKILL.md              # Claude Code skill definition (the AI prompt)
├── README.md             # This file
└── scripts/
    └── extract.py        # Document extraction engine (818 lines)
                          # Supports 10+ formats with auto-detection
                          # 4 extraction modes: text, technical, paddleocr, auto
```

## Comparison

| Feature | knowledge-to-notes | book-to-skill |
|---------|-------------------|---------------|
| Output | One beautiful `.md` note | Full skill directory (SKILL.md + chapters/ + glossary/...) |
| Use case | Read and copy-paste to Notion | Query via slash command in Claude Code |
| Design focus | Human readability (UX) | Machine query-ability (token efficiency) |
| PDF image OCR | ✅ PaddleOCR | ❌ (do_ocr=False by default) |
| Output location | Desktop (for easy access) | ~/.claude/skills/ (for agent use) |

## FAQ

**Q: Does it cost money?**
A: The Python extraction layer runs locally for free. The AI formatting step uses Claude Code tokens (typically 2K-10K tokens for most documents, which costs a few cents).

**Q: Can it handle Chinese documents?**
A: Yes. PaddleOCR PP-OCRv5 supports 109 languages, and PP-StructureV3 handles Chinese layout particularly well. Chinese text recognition accuracy is state-of-the-art.

**Q: What if my PDF has no text layer (pure images)?**
A: Use `--mode paddleocr` in extract.py, or let the skill auto-detect. PaddleOCR will OCR every page and extract the text, then the AI formats it. It's slower (5-15s per page) but produces excellent results.

**Q: Can I use this without Claude Code?**
A: You can use `extract.py` standalone for text extraction, but the AI formatting step requires a Claude-compatible agent. For non-agent use, you could adapt the SKILL.md prompt to work with the Claude API directly.

**Q: Where are the extracted files stored?**
A: Temporary files go to the system temp directory and are cleaned up after processing. The final note is saved to your Desktop.

## Contributing

This project is part of a family of document-processing skills. Found a bug or have a feature idea? Open an issue or PR.

## License

MIT — do whatever you want with it.

---

Built with ☕ and a strong belief that beautiful notes change how we learn.
