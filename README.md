# Awesome Claude Code — Paper Proofreading Prompts

Two prompts for proofreading LaTeX research papers with Claude Code, designed for robotics/CV conference and journal submissions (ICRA, RSS, CVPR, RA-L, T-RO level).

---

## Prompts

### [`prompts/01_latex_workspace_review.md`](prompts/01_latex_workspace_review.md)

**LaTeX infrastructure audit** — detects workspace-level errors before submission.

- Finds submission-blocker macros (author comment macros, revision highlights left active)
- Checks `hyperref` → `cleveref` load order
- Detects duplicate BibTeX keys, missing figure files, dummy/placeholder figures
- Finds hardcoded method names vs. `\methodname` macros
- Checks label naming conventions, duplicate labels, forward references
- **Two-phase**: audits and reports → user decides which to fix

### [`prompts/02_paper_proofreading.md`](prompts/02_paper_proofreading.md)

**Paper content proofreading** — detects language, clarity, scientific, and formatting issues in the paper itself.

- Acts as a strict ICRA/RSS/CVPR-level reviewer
- Catches non-native English patterns (e.g., "presented" → "demonstrated", "parallelly" → "in parallel")
- Checks scientific claims, logical gaps, overclaiming
- Reviews figure/table captions for self-containedness
- Checks LaTeX formatting (unit spacing, thousand separators, reference consistency)
- **Two-phase**: lists all issues first → user decides which to fix

---

## How to Use

### Option A — Paste directly into Claude Code

1. Copy the contents of the prompt file
2. Paste into your Claude Code session
3. Follow with your files (see input recommendations below)

### Option B — Use as a `@` reference

In Claude Code, reference the prompt file directly:

```
@prompts/01_latex_workspace_review.md

Please audit this workspace: @main.tex @shortcuts.tex
```

---

## What to Provide as Input

### For `01_latex_workspace_review.md`

Provide the root `.tex` file (whatever it is named) and your macro file:

```
@prompts/01_latex_workspace_review.md

@main.tex @shortcuts.tex
```

### For `02_paper_proofreading.md`

Provide **both** the `.tex` source files **and the compiled PDF**:

```
@prompts/02_paper_proofreading.md

@sections/introduction.tex @sections/methodology.tex @sections/experiments.tex
@paper.pdf
```

> **Why the PDF?**
> The compiled PDF lets Claude double-check two things that are invisible in the source alone:
> - **Figure placement** — whether figures are displaced far from their reference in the text (e.g., a figure floating to a different page or section)
> - **PDF-level annotations** — any comments or highlights left in the PDF from a previous review round that were not resolved

### Option C — Place in `CLAUDE.md`

Copy the relevant prompt into your paper project's `CLAUDE.md` to activate it automatically for that project.

---

## Design Principles

- **Detect first, fix later** — Claude lists all issues with numbers, user selects which to discard before any changes are made
- **Modular categories** — each review category is a labeled, self-contained block; reorder by moving blocks in the markdown
- **Real-world patterns** — rules extracted from actual annotated paper reviews across ICRA, RA-L, AAAI, BMVC, CVPR papers
- **Non-native writer aware** — includes specific patterns common in papers written by Korean/Japanese researchers

---

## Related

- [paper-writing-checklist](https://github.com/LimHyungTae/paper-writing-checklist) — LaTeX workspace guidelines this prompt references
