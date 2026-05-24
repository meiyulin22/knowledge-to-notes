---
name: knowledge-to-notes
description: "Transform messy notes, PDFs, and documents into beautifully formatted, Notion-ready markdown notes with rich emoji, clear hierarchy, and UX-driven layout. Preserves ALL source content with full depth — never silently drops sections. Auto-splits multi-topic sources into multiple files. Domain-agnostic: works for any subject (programming, math, history, biology, business, etc.)."
compatibility: "Claude Code skill directories (~/.claude/skills) and Amp skill directories (~/.config/agents/skills, ~/.config/amp/skills)"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
argument-hint: <path-to-document> [output-name]
---

# Knowledge to Notes (k2n) — v3

Transform raw documents into beautifully formatted, human-readable study notes optimized for Notion, Obsidian, or any markdown editor. **Beautiful AND complete** — never silently drops content. **Domain-agnostic** — every example below is illustrative only; the actual structure, topics, and terminology must be derived from the user's source document.

## Philosophy

Most notes are ugly. Dense walls of text crush motivation before reading begins. This skill treats note formatting as **UX design** — every heading, emoji, table, and callout serves a purpose: make the reader *want* to learn.

But beautiful is not the same as short. **The cardinal sin is silent omission** — dropping content from the source without telling the user. Visual polish should reveal structure, not hide content. A 60K-word source becomes a well-organized 50K-word note, not a 3K-word skim.

**Format for humans, not machines.** Visual hierarchy, strategic whitespace, emoji landmarks, and section-appropriate depth create flow state. The goal is notes so pleasant to read that users learn deeply *and* lose track of time.

## What It Does

1. **Inventory** every section in the source as ground truth
2. **Interview** user about output mode (速查卡 vs 学习材料) and depth
3. **Extract** text from any supported document
4. **Decide** single-file vs multi-file output based on topic count
5. **Format** every inventoried section with required fields (definition + example + scenario + trap)
6. **Verify** every inventory item appears in output before delivery
7. **Report** coverage stats — warn if compression > 70%

## Usage

```
/knowledge-to-notes <path-to-document> [output-name]
```

**Output:**
- Single topic → `~/Desktop/<name>-知识笔记.md`
- Multi-topic → `~/Desktop/<name>-知识笔记/` directory with `00-overview.md` + topic files

---

## Step 0 — Source Inventory (REQUIRED, before anything else)

**This is the ground truth. Skipping it = silent omission later.**

After extraction (Step 3), but before any formatting, build a complete tree of all top-level and second-level sections **as they actually appear in the source document**. Do not invent topics. Do not omit topics. Write to `<tempdir>/k2n_work/inventory.md`.

### Format (abstract — fill with topics from THE ACTUAL source)

```markdown
- [ ] <Top-level Topic A as written in source>
  - [ ] <Subtopic A.1>
  - [ ] <Subtopic A.2>
  - [ ] <Subtopic A.3>
- [ ] <Top-level Topic B>
  - [ ] <Subtopic B.1>
  - [ ] <Subtopic B.2>
- [ ] <Top-level Topic C>
  - [ ] ...
```

Topics are whatever the source covers. If the source is a Python tutorial, topics might be `数据类型 / 控制流 / 函数 / 类 / 装饰器 / 异步`. If it's a history book, topics might be `先秦 / 秦汉 / 三国 / 隋唐 / 宋元`. If it's a biology textbook, topics might be `细胞结构 / 代谢 / 遗传 / 进化`. **Read the source headings; do not pattern-match to a domain you've seen before.**

### Rules

- One inventory item per top-level heading and per meaningful subheading in the source
- Use the source's original wording (do not translate or rephrase)
- If the source has no explicit headings, derive topics from natural section breaks but mark them: `(derived)`
- **Do not proceed to Step 5 until this file is written.** Step 7.5 verifies output against this list.

### Long-document handling (>20K words)

If extracted text exceeds 20K words OR inventory has >40 leaf items:
- **Process section-by-section, writing each output file incrementally** — do not try to format everything in one response
- After writing each section, mark its inventory entry `[x]` and continue
- This is mandatory; reading 60K words once and writing from memory **always** drops content

---

## Step 1 — Validate Input

```bash
test -f "$DOC_PATH" && echo "FILE_OK" || echo "FILE_NOT_FOUND"
```

If not found or unsupported format, report error and list supported formats: PDF, EPUB, DOCX, TXT, MD, HTML, RTF, MOBI/AZW. Image-based PDFs / PNG / JPG require the `full` branch with PaddleOCR.

---

## Step 2 — Goal Interview (MANDATORY)

Ask the user, all three required (but accept "默认" / "default" as a one-word answer to skip):

> 📋 **Before I start, 3 quick questions:**
>
> **1. Output mode?**
>   - `速查卡` — minimalist, scan-and-recall (think: cheat sheet)
>   - `学习材料` — complete, learn-from-zero ✅ **default**
>   - `教程改造` — restructure scattered content into linear chapters
>
> **2. Depth?**
>   - `浅` — concept + 1 example
>   - `中` — concept + multiple examples + common traps ✅ **default**
>   - `深` — add internals, comparisons, bonus knowledge
>
> **3. Background (optional but improves quality):**
>   - Prior knowledge?
>   - Focus area? (exam prep / practical use / first principles / curiosity)

Save answers to `<tempdir>/k2n_work/goals.md`. Use these to calibrate every following step.

**Do not skip this step** — `速查卡` and `学习材料` produce radically different outputs. Guessing wrong wastes everyone's time.

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

# Run extraction — pick mode by file:
#   PDFs that may be image-based → paddleocr (full branch only)
#   Text PDFs → text
#   MD/TXT/EPUB/DOCX → text (auto-detected)
"$PYTHON_BIN" "$SCRIPT_PATH" "$DOC_PATH" --mode $EXTRACTION_MODE --install-missing yes
```

This produces:
- `<tempdir>/k2n_work/full_text.txt` — extracted text
- `<tempdir>/k2n_work/metadata.json` — stats

Read the extracted text in full **before** building the inventory.

---

## Step 4 — Analyze Structure

Read the extracted text and identify:
- **Main topic(s)** and subtopics
- **Natural section breaks** (headings, chapter markers, topic shifts)
- **Key entities**: terms, definitions, formulas, names, dates — whatever the domain emphasizes
- **Patterns**: comparisons, sequences, hierarchies, cause-effect
- **Code blocks / diagrams / formulas / quotes** to preserve verbatim
- **Difficulty spikes**: complex sections that need extra clarity

Now build the inventory (Step 0).

---

## Step 4.5 — Relevance Check (CONDITIONAL)

**Only run this if the source contains genuinely outdated or superseded content.** Examples across domains:
- Programming: deprecated APIs, old framework versions, legacy syntax
- Science: theories that have been revised or replaced
- Business / law: regulations that have been amended
- General: historical practices no longer in use

If the source is all current, **skip this step** — the table is noise.

When applicable, output:

```markdown
📊 **Learning Priority: What's worth your time?**

| Item | Current Status | Time Investment |
|------|---------------|-----------------|
| ... | ✅ Core / ⚠️ Legacy / ❌ Superseded | Deep dive / Skim / Skip |
```

- ✅ **Core** → full treatment per Step 6
- ⚠️ **Legacy / Superseded** → name the modern replacement; minimal detail; no extensive examples
- Place this table at the top of `00-overview.md` (multi-file) or before the first chapter (single-file)

---

## Step 5 — Output Mode Decision

Count distinct top-level topics in the inventory. Apply this rule:

| Condition | Output |
|-----------|--------|
| Single topic, <6 sections, source <20K chars | Single file: `~/Desktop/<name>-知识笔记.md` |
| ≥2 distinct domains/subjects | **Directory** (see below) |
| Single topic but >40 leaf sections OR source >20K words | **Directory** by chapter |

### Directory layout (multi-file, abstract)

```
~/Desktop/<name>-知识笔记/
├── 00-overview.md         # 学习路线 + Learning Priority + 章节索引
├── 01-<topic-a-slug>.md
├── 02-<topic-b-slug>.md
├── 03-<topic-c-slug>.md
├── ...
└── 99-quickref.md         # optional: aggregated quick-reference tables
```

Slugs come from the inventory — kebab-case, lowercased, ASCII when possible. The numeric prefix reflects reading order, not source order (use Step 2's "教程改造" mode if reordering for pedagogy).

`00-overview.md` must contain:
- A short intro (2-3 sentences) explaining what this collection covers
- Learning Priority table (if Step 4.5 applies)
- 学习路线: linear order to read the files, with a 1-sentence "why this order"
- 章节索引: relative markdown links to every other file with a 1-line description

**Never cram multiple distinct domains/subjects into one file.** N independent topics → N files (plus overview).

---

## Step 6 — Format with UX Principles

### Visual Hierarchy
- `#` H1 = document/topic title (one per file, with emoji prefix appropriate to domain)
- `##` H2 = major sections (with emoji landmarks)
- `###` H3 = sub-sections
- `####` H4 = detail points (use freely to break up long content — better than truncating)

### Emoji Landmarks (semantic, not decorative)
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
🔧 → API / method / tool reference
⚡ → Performance / pitfall
🧪 → Experiment / proof / derivation
📜 → Quote / primary source / citation
```

Pick emoji that fit the domain. A history note will lean on `⏱ 📜 🔄`; a math note on `🧪 📊 💡`; a programming note on `🔧 ⚡ ⚠️`.

### REQUIRED fields per concept (学习材料 mode)

For **every** concept in the inventory, the output **must** include all of these (in this order):

1. ✅ **One-sentence definition** — what is it
2. ✅ **Concrete example** — preserved from source verbatim if present (see Verbatim Preservation Rule)
3. ✅ **Use case / context** — when does this matter, drawn from source where possible
4. ✅ **Common trap / FAQ / edge case** — at least one
5. ✅ **Comparison with related concepts** — where applicable

The "comparison" field is domain-flexible. Examples across domains:
- Programming: `var` vs `let` vs `const`; sync vs async; pass-by-value vs by-reference
- Math: derivative vs integral; vector vs scalar
- Biology: mitosis vs meiosis; prokaryote vs eukaryote
- Business: fixed cost vs variable cost; B2B vs B2C
- History: cause vs trigger; revolution vs reform

If the concept genuinely has no related concept to compare against, write "—" and move on.

In `速查卡` mode, only #1, #2, #4 are required.

### 🔒 Verbatim Preservation Rule

**Any concrete artifact from the source must be reproduced verbatim** — code blocks, formulas, quotes, dates, statistics, named entities.
- ✅ You may add comments, captions, or labels
- ✅ You may add a NEW artifact alongside (e.g. a "modern equivalent" or "related example")
- ❌ You may NOT shorten, paraphrase, or "clean up" source artifacts
- ❌ You may NOT pick "the best 2 of 5 examples" — keep all 5
- ❌ You may NOT replace source examples with your own unless the source's example is broken (then mark with ❗ Correction)

If the source has 5 examples of a concept (in any domain — code, problem sets, case studies, historical events), output all 5.

### Section length

- A standalone concept gets ≥10 lines of explanation + concrete artifact + table when in 学习材料 mode
- When prose runs >15 lines, insert an H4 sub-heading or callout — **break it up, do not truncate**
- Tables can be any length; no row caps

### Content Blocks (use liberally)
```markdown
> 💡 **Key Insight:** <one-sentence revelation>

> ⚠️ **Common Trap:** <what people get wrong>
> ❌ <wrong approach>
> ✅ <right approach>

> 💎 **TL;DR:** <one paragraph that captures everything>
> NOTE: TL;DR is a SUMMARY of detail above, never a SUBSTITUTE for it

> 🧠 **Memory Hook:** <mnemonic or vivid association>

> 📝 **Action:** <something to try, practice, or solve>
```

### Tables
- **Cover ALL items in the source** — no "max 3-6 rows" cap
- If a comparison table would have 30 rows, output 30 rows (split with sub-headings if needed)
- Use emoji in headers for visual scanning
- Keep tables under 6 columns wide for readability (rows have no cap, columns do)

### Code Blocks (programming/technical content)
- Always specify language: ` ```python`, ` ```javascript`, ` ```bash`, ` ```sql`, ` ```rust`, etc.
- Length follows source — do not truncate working examples
- Add a one-line comment at top only if the example's purpose isn't obvious

### Section Endings

Each major section (H2) must end with **both**:
1. A **Quick-Reference Table** covering all items in that section (no row cap)
2. A `> 💎 **Section Summary:**` callout (1-3 sentences, optional for short sections)

---

## Step 7 — Write Output

Write according to Step 5 decision:
- **Single file:** `~/Desktop/<output-name>-知识笔记.md`
- **Multi-file:** `~/Desktop/<output-name>-知识笔记/` directory

If `output-name` not provided, derive from filename (strip extension, replace underscores/hyphens with spaces).

For long documents being processed section-by-section, write each file incrementally and tick off inventory entries as you go. **Do not** hold the whole output in memory and write at the end.

---

## Step 7.5 — Coverage Verification (REQUIRED before delivery)

For every leaf entry in `inventory.md`:
1. Grep the output file(s) for the section name or its key terms
2. ✅ Found → mark `[x]` in inventory
3. ❌ Missing → must add the section before delivery, do not skip

```bash
# Example check (adjust paths)
INV="<tempdir>/k2n_work/inventory.md"
OUT_GLOB=~/Desktop/<output>-知识笔记*
grep -oE '\- \[ \] .+' "$INV" | sed 's/^- \[ \] //' | while read section; do
  if ! grep -rq "$section" $OUT_GLOB; then
    echo "MISSING: $section"
  fi
done
```

Compute compression ratio: `output_words / source_words`.
- **>30%** (kept ≥30% of content) → normal in 学习材料 mode
- **<30% in 学习材料 mode** → ⚠️ likely silent omission, re-check inventory
- **<10% in any mode except 速查卡** → ❌ red flag, refuse to deliver until expanded or user confirms 速查卡 mode

---

## Step 8 — Report

```
✅ Note created at: ~/Desktop/<name>-知识笔记[.md|/]

📄 Source: <filename> (<format>, <N>K words)
📊 Coverage: <X>/<Y> sections from inventory
📁 Files: <list of files if multi-file>
📈 Source → Output: <source-words> → <output-words> words (kept <pct>%)

<warning lines if compression >70% or coverage <100%>

Open the file(s), copy content, paste into Notion / Obsidian / your editor.
```

---

## Annotation System

Use these markers consistently to guide the reader's attention:

| Marker | Meaning | Treatment |
|--------|---------|-----------|
| ⭐ **Key Point** | High-frequency / exam-critical / practical must-know | **Expand**: extra examples, diagrams, common questions. Never summarize. |
| ⚠️ **Outdated** | Old approach replaced by modern alternatives | One summary table, name the modern replacement. Minimal detail; never enumerate. |
| 💡 **Bonus** | Worth knowing but not in original notes | Brief but substantive — 1-2 paragraphs OK if it genuinely helps understanding |
| ❗ **Correction** | Original notes have a factual error | Show original → corrected, explain why the original was wrong |

⭐ and ⚠️ are **opposites**: ⭐ expands, ⚠️ contracts. Apply this asymmetry deliberately.

---

## Quality Rules

1. **🚫 Cardinal sin: silent omission** — never drop a section without an explicit `⚠️ Outdated` annotation. Step 7.5 verification is mandatory.
2. **🔒 Preserve source artifacts verbatim** — code, formulas, quotes, examples; comments may be added, lines may not be removed.
3. **📋 Required fields per concept** — definition + example + scenario + trap + comparison (in 学习材料 mode).
4. **📁 Multi-domain sources → multiple files** — N distinct domains = N files minimum, never one.
5. **📏 Bite-sized but complete** — break long blocks with sub-headings, do not truncate content.
6. **📊 Tables cover ALL items** — no "max 6 rows" cap; split with sub-headings if needed.
7. **🎯 Front-load value** — most important insight in the first section of each file.
8. **🌐 Language matches source** — output in the same language as the input; keep terminology, code, and proper nouns in their original form.
9. **🚫 No preamble** — output the note content directly, no "Here's your formatted note" intros.
10. **✅ Verify before delivery** — Step 7.5 is not optional.
11. **🧭 Domain-agnostic** — adapt structure, emoji, and field semantics to the source's domain. Never force a programming-shaped output onto a history source, or vice versa.

---

## Anti-patterns (don't do these)

| ❌ Don't | ✅ Do |
|---------|-------|
| Read the whole 60K-word source and write from memory | Inventory first, then process section-by-section, writing incrementally |
| Replace detail with a TL;DR | TL;DR **plus** detail; TL;DR summarizes, never substitutes |
| Cap a table at 6 rows when source has 30 | Output all 30, split with sub-headings if needed |
| Cram multiple distinct domains into one file | One file per domain |
| Skip a "basic" section because it's obvious | Include it; the reader can skim, but it must be present |
| Rewrite source artifacts "more cleanly" | Keep verbatim; add comments only |
| Say "see source for details" instead of writing details | The whole point is to format the details — write them |
| Ask for goals/depth as optional and assume defaults | Ask explicitly; only treat user's "default" reply as opt-in to defaults |
| Treat the Learning Priority table as universal | Skip it for sources without legacy/superseded content |
| Pattern-match topics from a domain you've seen before | Read the source's actual headings; use the source's actual terminology |
