# Knowledge to Notes (k2n)

> **Transform messy documents into stunning, Notion-ready study notes.**
> 🖼️ **Have scanned PDFs, images, or documents with pictures?** Switch to the **[full](https://github.com/meiyulin22/knowledge-to-notes/tree/full)** branch — it adds built-in OCR for anything with images: scanned books, photos, screenshots, image-only PDFs, and more.

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

1. **Extracts** text from PDF, EPUB, DOCX, Markdown, HTML, TXT, RTF, MOBI/AZW
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
pip install PyPDF2 pdfminer.six

# Or install only what you need:
# For PDF:       pip install PyPDF2 pdfminer.six
# For EPUB:      pip install ebooklib beautifulsoup4
# For DOCX:      pip install python-docx
# For DOCX:      pip install python-docx
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
| **PDF** | pdftotext / PyPDF2 / pdfminer | Text-based PDFs |
| **Markdown / TXT** | Direct read | Instant |
| **EPUB** | ebooklib + BeautifulSoup4 | Best quality |
| **DOCX** | python-docx | Tables preserved |
| **HTML** | BeautifulSoup4 | Clean text extraction |
| **RTF** | striprtf | - |
| **MOBI / AZW** | Calibre ebook-convert | Requires Calibre installed |

> 🔍 **Need image support (scanned PDFs, photos, screenshots)?** Switch to the **[full](https://github.com/meiyulin22/knowledge-to-notes/tree/full)** branch.

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────────┐
│  PDF/MD/    │────▶│  extract.py  │────▶│  Claude Code  │────▶│  Beautiful │
│  EPUB/DOCX  │     │  text/OCR    │     │  Analyze +    │     │  .md note  │
│             │     │  extraction  │     │  Format       │     │  on Desktop │
└─────────────┘     └──────────────┘     └───────────────┘     └────────────┘
     Input           Python (free)        AI (token cost)        Output
```

- **extract.py**: Python script that auto-detects format and picks the best extraction method (pdftotext, PyPDF2, ebooklib, python-docx, etc.). All extraction runs locally — no API calls.
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
| PDF extraction | pdftotext (poppler), PyPDF2, pdfminer.six |
| EPUB extraction | ebooklib + BeautifulSoup4 |
| DOCX extraction | python-docx |
| HTML extraction | BeautifulSoup4 |
| RTF extraction | striprtf |
| MOBI/AZW extraction | Calibre ebook-convert |

## Project Structure

```
knowledge-to-notes/
├── SKILL.md              # Claude Code skill definition (the AI prompt)
├── README.md             # This file
└── scripts/
    └── extract.py        # Document extraction engine
                          # Supports 7+ formats with auto-detection
```

## Comparison

| Feature | knowledge-to-notes | book-to-skill |
|---------|-------------------|---------------|
| Output | One beautiful `.md` note | Full skill directory (SKILL.md + chapters/ + glossary/...) |
| Use case | Read and copy-paste to Notion | Query via slash command in Claude Code |
| Design focus | Human readability (UX) | Machine query-ability (token efficiency) |
| Output location | Desktop (for easy access) | ~/.claude/skills/ (for agent use) |
| PDF image OCR | Via [full](https://github.com/meiyulin22/knowledge-to-notes/tree/full) branch | ❌ (do_ocr=False by default) |

## FAQ

**Q: Does it cost money?**
A: The Python extraction layer runs locally for free. The AI formatting step uses Claude Code tokens (typically 2K-10K tokens for most documents, which costs a few cents).

**Q: What if my PDF has no text layer (pure images)?**
A: This branch handles text-based documents only. For image-based PDFs and scanned documents, switch to the **[full](https://github.com/meiyulin22/knowledge-to-notes/tree/full)** branch — it adds PaddleOCR for image recognition.

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
