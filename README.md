<div align="center">
    <h1>Awesome Claude Code — Paper Proofreading</h1>
    <a href="https://github.com/LimHyungTae/awesome-claudecode-paper-proofreading"><img src="https://img.shields.io/badge/Claude_Code-Prompt-blueviolet?logo=anthropic" /></a>
    <a href="https://github.com/LimHyungTae/awesome-claudecode-paper-proofreading"><img src="https://img.shields.io/badge/LaTeX-Workspace_Audit-008080?logo=latex&logoColor=white" /></a>
    <a href="https://github.com/LimHyungTae/awesome-claudecode-paper-proofreading"><img src="https://img.shields.io/badge/Venue-ICRA%20%7C%20RSS%20%7C%20CVPR%20%7C%20RA--L%20%7C%20T--RO-blue" /></a>
    <br />
    <br />
    <!-- VIDEO PLACEHOLDER: replace the line below with your demo video embed -->
    <!-- <p align="center"><a href="YOUR_VIDEO_URL"><img src="YOUR_THUMBNAIL_URL" alt="Demo Video" width="95%"/></a></p> -->
    <p><strong><em>Detect first. Fix with confidence.</em></strong></p>
</div>

______________________________________________________________________

## :rocket: Overview

Two Claude Code prompts for rigorous proofreading of LaTeX research papers, designed for robotics and computer vision submissions at ICRA, RSS, CVPR, RA-L, and T-RO level.

The workflow is **two-phase**: Claude detects and lists all issues with unique numbers `[1]`, `[2]`, `[3]`... — then waits. The user selects which issues to fix or discard before any file is modified.

______________________________________________________________________

## :page_facing_up: Prompts

### [`prompts/01_latex_workspace_review.md`](prompts/01_latex_workspace_review.md)

**LaTeX infrastructure audit** — detects workspace-level errors before submission.

| Check | Description |
|-------|-------------|
| C1 | Preamble configuration (`hyperref`, `cleveref`, `caption` setup) |
| C2 | Package load order & conflicts |
| C3 | Macro safety, `\methodname` consistency, subscript macros, `\etalcite` |
| C4 | Cross-reference consistency (multi-ref, subcaption `Fig. 5(a)` format) |
| C5 | Label naming conventions, duplicate labels |
| C6 | Citation & bibliography integrity, duplicate BibTeX keys |
| C7 | Figure & table safety (missing files, dummy figures, label placement) |
| C8 | Hidden human errors (TODOs, inconsistent naming, `\vspace` hacks) |
| C9 | Academic writing patterns detectable in source (`\ie`, `\eg`, units) |

### [`prompts/02_paper_proofreading.md`](prompts/02_paper_proofreading.md)

**Paper content proofreading** — acts as a strict conference-level reviewer.

| Category | Description |
|----------|-------------|
| A | Language & grammar, tense consistency, Related Work tense |
| B | Non-native English patterns ("parallelly", "presented", "suggest") |
| C | Scientific clarity: overclaiming, "significantly", unsupported claims |
| D | Structure & flow: intro claims, experiment purpose, equation narrative |
| E | Figure/table/caption review: self-containedness, reference order, quantitative consistency |
| F | LaTeX formatting: units, thousand separators, `\ie`/`\eg` macros |
| G | Abstract (WHY→PROBLEM→HOW→RESULTS) & conclusion quality |
| H | Notation consistency: symbol overload, boldface vectors, coordinate frames |
| I | Hyphenation: compound adjectives, `-ly` adverb rule |

______________________________________________________________________

## :hammer: How to Use

### Step 1 — Run the workspace audit first

Provide the root `.tex` file and macro file:

```
@prompts/01_latex_workspace_review.md

@main.tex @shortcuts.tex
```

### Step 2 — Run the content proofreader

Provide both the `.tex` source files **and the compiled PDF**:

```
@prompts/02_paper_proofreading.md

@sections/introduction.tex @sections/methodology.tex @sections/experiments.tex
@paper.pdf
```

> **Why the PDF?**
> The compiled PDF lets Claude cross-check two things invisible in source alone:
> - **Figure placement** — figures displaced far from their in-text reference
> - **PDF-level annotations** — leftover review comments not yet resolved

### Step 3 — Decide which issues to fix

After Phase 1 output, respond with:

```
discard 3, 7, 12    ← skip specific issues
fix all critical    ← fix only CRITICAL issues
proceed with all    ← fix everything
```

______________________________________________________________________

## :bulb: Design Principles

- **Detect first, fix later** — Claude never modifies files until the user explicitly confirms
- **Modular categories** — each check is a labeled block; reorder or disable by editing the table at the top of each prompt
- **Real-world patterns** — rules extracted from annotated paper reviews across ICRA, RA-L, AAAI, BMVC, and CVPR submissions
- **Non-native writer aware** — covers patterns common in papers by Korean/Japanese researchers
- **`figures/` vs `pics/`** — distinguishes final manuscript figures from raw image sources

______________________________________________________________________

## :link: Related

- [paper-writing-checklist](https://github.com/LimHyungTae/paper-writing-checklist) — LaTeX workspace guidelines referenced by these prompts
