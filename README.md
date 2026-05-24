# Knowledge to Notes (k2n)

> **Transform messy documents into stunning, Notion-ready study notes.**
>
> 🤖 **Using a multimodal LLM host** (Claude Code, Cursor with Claude/GPT-4o, Gemini)? You're on the right branch — your model reads images natively, no OCR needed.
>
> 🔌 **Using a text-only LLM host** (DeepSeek, older local models without vision)? Switch to the **[full](https://github.com/meiyulin22/knowledge-to-notes/tree/full)** branch — it bolts PaddleOCR on as an external OCR plugin so your text-only model can still handle image PDFs.

Tired of dense PDF walls of text that kill your motivation before you even start reading? Knowledge to Notes turns raw documents — PDFs, Markdown files, EPUBs, Word docs — into beautifully formatted markdown notes with rich visual hierarchy, strategic emoji landmarks, and bite-sized chunks that make learning addictive.

## What Problem This Solves

| Before | After |
|--------|-------|
| Ugly, dense walls of text | Clean, scannable sections with emoji landmarks |
| No visual hierarchy | Multi-level headings, tables, and callouts |
| Hard to find key points | Insights, traps, and summaries highlighted |
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
# For PDF:    pip install PyPDF2 pdfminer.six
# For EPUB:   pip install ebooklib beautifulsoup4
# For DOCX:   pip install python-docx
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

> 🔍 **Running on a text-only LLM** (DeepSeek, etc.) and need image PDF support? Switch to the **[full](https://github.com/meiyulin22/knowledge-to-notes/tree/full)** branch — it adds PaddleOCR as an external plugin.

## Design Philosophy

> Format for humans, not machines.

Every design choice serves one goal: make the reader *want* to keep reading.

- **Emoji landmarks** create instant visual navigation (not decoration)
- **Bite-sized chunks** prevent cognitive overload (no paragraph over 4 lines)
- **Front-loaded insights** give immediate value (most important point first)
- **Summaries** let you grasp a topic in 10 seconds
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
```

## FAQ

**Q: Does it cost money?**
A: The Python extraction layer runs locally for free. The AI formatting step uses Claude Code tokens (typically 2K-10K tokens for most documents).

**Q: What if I have scanned/image-based PDFs?**
A: If your LLM host is multimodal (Claude, GPT-4o, Gemini), this branch already handles it — the model reads page images directly. If your host is text-only (DeepSeek, older local models), switch to the **[full](https://github.com/meiyulin22/knowledge-to-notes/tree/full)** branch — it bolts PaddleOCR on as an external OCR plugin.

**Q: Can I use this without Claude Code?**
A: You can use `extract.py` standalone for text extraction, but the AI formatting step requires a Claude-compatible agent.

## License

MIT — do whatever you want with it.

---

Built with ☕ and a strong belief that beautiful notes change how we learn.
