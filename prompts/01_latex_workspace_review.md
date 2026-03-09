# LaTeX Workspace Review Prompt for Claude Code

## Purpose

You are a LaTeX research-paper workspace reviewer.
Your goal is to detect potential human errors in the LaTeX workspace before submission.

This repository follows the guidelines at:
https://github.com/LimHyungTae/paper-writing-checklist

> **Do NOT rewrite or fix anything during Phase 1.**
> **Do NOT modify any files until the user confirms.**

---

## How This Prompt Works (Two-Phase)

**Phase 1 — Audit:** Run all active checks. Output every issue with a unique number `[1]`, `[2]`, `[3]`...

**Phase 2 — Fix:** After the user specifies which issues to fix, apply only the approved ones.

---

## Files to Inspect

Read the following files if they exist (adapt to the actual workspace structure):

**Main document:**
- The root `.tex` file that contains `\begin{document}` (the filename varies per user — find it by looking for the file with `\documentclass` and `\begin{document}`)

**Macro files (check all that exist):**
- `shortcuts.tex`
- `macros.tex` / `commands.tex`
- `preamble.tex`

**Shared preamble (if workspace uses a shared references folder):**
- `../references/preamble_packages.tex`
- `../references/preamble_symbols.tex`

**Section files:**
- `sections/*.tex`

**Bibliography:**
- `*.bib`
- `../references/*.bib`

**Figures and image sources (two distinct folders):**
- `figures/` — contains the **final figures inserted into the manuscript** via `\input{}` or `\includegraphics{}`. Check that every file referenced in `.tex` exists here.
- `pics/` — contains **raw image sources** (original photos, screenshots, SVGs, raw exports). Files here are not directly inserted into the manuscript. Do NOT flag missing `.tex` references for files in `pics/`.

**Deprecated / leftover:**
- `deprecated/` folder contents — flag anything still `\input`-ted from the main document

---

## Active Review Categories

> **To reorder or disable a check: move the row or delete it.**

| Order | Check ID | Check Name                           | Enabled |
|-------|----------|--------------------------------------|---------|
| 1     | C1       | Preamble Configuration               | ✅      |
| 2     | C2       | Package Load Order & Conflicts       | ✅      |
| 3     | C3       | Macro Safety & Naming Consistency    | ✅      |
| 4     | C4       | Cross-Reference Consistency          | ✅      |
| 5     | C5       | Label Naming Convention              | ✅      |
| 6     | C6       | Citation & Bibliography              | ✅      |
| 7     | C7       | Figure & Table Safety                | ✅      |
| 8     | C8       | Hidden Human Errors                  | ✅      |
| 9     | C9       | Academic Writing (LaTeX-Detectable)  | ✅      |

---

## Review Rules

---

### CHECK C1 — Preamble Configuration

Verify the preamble (whether in the root `.tex` file, `preamble.tex`, or a shared file):

- **Duplicate packages** — same package loaded more than once across all input files
- **Conflicting packages** — packages known to be incompatible
- **Unused packages** — loaded but no corresponding command used anywhere
- **Missing packages** — custom macros reference a package not loaded

**Verify these specific configurations:**

**cleveref setup:**
```
✅  \usepackage[nameinlink,capitalize]{cleveref}
⚠️  \usepackage[capitalize]{cleveref}       ← missing nameinlink (hyperlinks only the number, not "Figure")
❌  \usepackage{cleveref}                   ← missing both options
```

Check `\Crefname` and `\crefname` customizations for:
- `section` → `Sec.` / `Secs.`
- `figure` → `Fig.` / `Figs.`
- `table` → `Table` / `Tables`
- `equation` → check if suppressed to `()` style (common for IEEE)
- `algorithm` → `Alg.` / `Algs.`

**hyperref setup:**
```
✅  \usepackage[colorlinks=true, allcolors=blue]{hyperref}
⚠️  Missing bookmarks=true for PDF reader navigation
```

**caption setup:**
```
✅  \captionsetup[figure]{font=footnotesize}
✅  \captionsetup[table]{font=footnotesize}
⚠️  Check if labelformat, labelsep, and name are consistent across figure and table
```

**math packages:**
- `amsmath` present?
- `mathtools` present? (check if loaded after `amsmath`)
- `bm` present for bold math?

**algorithm environments:**
- If algorithms appear in sections, check `algorithm2e` or `algorithmicx` is loaded
- Check for `algorithm` + `algorithmic` vs `algorithm` + `algpseudocode` conflicts

---

### CHECK C2 — Package Load Order & Conflicts

Known critical orderings to verify:

| Rule | Correct Order |
|------|--------------|
| cleveref after hyperref | `hyperref` → then → `cleveref` |
| mathtools after amsmath | `amsmath` → then → `mathtools` |
| xcolor before tikz | `xcolor` → then → `tikz` (if both present) |
| caption before subcaption | `caption` → then → `subcaption` |

```
❌  \usepackage{cleveref}
    \usepackage{hyperref}    ← WRONG: cleveref before hyperref breaks links

✅  \usepackage{hyperref}
    \usepackage{cleveref}    ← CORRECT
```

Also flag:
- `\usepackage{times}` or `\usepackage{pslatex}` — often causes issues; flag if present
- `subfig` and `subcaption` loaded simultaneously (conflict)
- `epsfig` loaded but `graphicx` also present (epsfig is a thin wrapper; redundant)

---

### CHECK C3 — Macro Safety & Naming Consistency

Inspect all custom macros in `shortcuts.tex`, `macros.tex`, or `commands.tex`.

**Method and dataset naming macros** (most common submission error):

Look for macros like `\methodname`, `\ours`, `\method`, `\datasetname` and verify:

1. Defined **exactly once** across all files
2. Used with **consistent capitalization** throughout all `.tex` files
3. Not mixed with hardcoded string versions

```
✅  \newcommand{\methodname}{FooNet\xspace}
    ...
    \methodname achieves...   ← correct

❌  \newcommand{\methodname}{FooNet\xspace}
    ...
    FooNet achieves...        ← hardcoded; breaks if name changes
    Foonet achieves...        ← wrong capitalization
```

**`\xspace` check:**
- Macros without `\xspace` that are used in running text may cause missing space after the macro
- `\newcommand{\method}{FooNet}` → should be `\newcommand{\method}{FooNet\xspace}`

**Macro redefining existing commands:**
- Flag any `\renewcommand` usage — verify it is intentional, not accidental
- Flag macros named identically to standard LaTeX or package commands

**Math-mode safety:**
- Macros used in both text and math mode — check for `\text{}` wrappers where needed
- Macros that use text-only commands (`\textbf`, `\emph`) inside math expressions

**Shared symbol conflicts:**
- If `../references/preamble_symbols.tex` exists, check for symbol name conflicts with locally defined macros in `shortcuts.tex`
- Duplicate `\newcommand` definitions across the shared and local files

**Subscript consistency in math expressions:**

Hardcoded subscript strings are a common source of inconsistency. If the same subscript label appears in multiple equations as a raw string, it should be a macro defined in `preamble_symbols.tex`.

Flag any subscript that is written as a literal string rather than a macro:
```
❌ _{\text{pred}}, _{\mathrm{pred}}   ← hardcoded; if used in 3+ places, define a macro
❌ _{\text{GT}}, _{\mathrm{GT}}
❌ _{\text{obs}}, _{\text{proj}}

✅ \newcommand{\subPred}{\mathrm{pred}}   ← defined in preamble_symbols.tex
   _{\subPred}                            ← used consistently
```

For each flagged subscript string, suggest the macro name and the line to add to `preamble_symbols.tex`.

---

### CHECK C4 — Cross-Reference Consistency

**Reference command consistency** — pick one style and use it everywhere:

```
❌ "Fig. 3 shows..."             ← hardcoded; breaks if figure is renumbered
❌ "Figure~\ref{fig:x} shows..." ← manual type label; inconsistent
✅ "\Cref{fig:x} shows..."       ← preferred with cleveref
```

**Multiple references** — when citing more than one figure, table, or equation together:

```
❌ "\Cref{fig:xx} and \Cref{fig:xy}"   ← produces "Fig. 1 and Fig. 2" (redundant label)
✅ "\Cref{fig:xx,fig:xy}"              ← cleveref auto-produces "Figs. 1 and 2"
✅ "Figs.~\ref{fig:xx} and~\ref{fig:xy}"  ← acceptable if not using cleveref list syntax
```

Flag any pattern where the same type label (`Fig.`, `Table`, `Eq.`) is repeated for each item in a list — it should appear only once before the group.

**Subcaption references** — when referencing a subfigure `(a)`, `(b)`, etc.:

```
❌ "Fig. 1a"       ← no parentheses; ambiguous
❌ "Fig. 1-a"      ← incorrect separator
✅ "Fig. 1(a)"     ← correct: parentheses around the subfigure label
✅ "\Cref{fig:x}(a)"  ← with cleveref for the parent + manual subfigure label
```

If `subcaption` or `subfig` is used, check whether `\subref{fig:x:a}` is configured to produce `(a)` and that the surrounding reference produces `Fig. 1(a)` not `Fig. 1a`.

**General checks:**
- Mixed use of `\Cref`, `\cref`, `\ref` — standardize
- Manual `"Fig."`, `"Table"`, `"Eq."`, `"Sec."` references in text instead of `\Cref{}`
- `\eqref{}` vs `\Cref{eq:}` — verify consistent choice
- References to figures that appear **before** the figure definition in the document (forward references for figures should be minimized)
- References to equations **before** the equation is defined
- References to sections that do not exist
- `~\Cref{}` vs `\Cref{}` — check spacing before cross-references

---

### CHECK C5 — Label Naming Convention

Verify all `\label{}` values follow a consistent prefix scheme:

| Type      | Expected Prefix | Example            |
|-----------|-----------------|--------------------|
| Figure    | `fig:`          | `fig:overview`     |
| Table     | `tab:`          | `tab:results`      |
| Equation  | `eq:`           | `eq:loss`          |
| Section   | `sec:`          | `sec:method`       |
| Algorithm | `alg:`          | `alg:main`         |
| Appendix  | `app:`          | `app:proof`        |

Flag:
- Labels without a typed prefix (`\label{overview}` instead of `\label{fig:overview}`)
- **Duplicated labels** — same label string defined in two places (causes LaTeX `multiply-defined` warning and wrong references)
- Labels defined but **never referenced** anywhere
- `\ref{}` calls referencing labels that do not exist

---

### CHECK C6 — Citation & Bibliography

**Citation integrity:**
- Citations used in text (`\cite{key}`) but missing from all `.bib` files
- Entries in `.bib` files but **never cited** in any `.tex` file
- **Duplicate BibTeX keys** — same key defined in multiple `.bib` files (e.g., `myRefs.bib` and `refs.bib`)

**BibTeX entry quality:**
- arXiv papers formatted inconsistently — some as `@misc`, some as `@article`
- Missing required fields: `author`, `title`, `year`, and either `journal` or `booktitle`
- Venue abbreviations inconsistent across entries (e.g., `CVPR` vs `Proc. CVPR` vs `IEEE CVPR`)
- Typos in URL fields (e.g., `Avaliable:` instead of `Available:`)

**Non-breaking space before citations:**
```
❌ " \cite{x}" or ".\cite{x}"   ← missing non-breaking space
✅ "~\cite{x}"                   ← correct
```

---

### CHECK C7 — Figure & Table Safety

**Figures:**

- Figure file referenced in `\includegraphics{}` or `\input{}` but **file does not exist** in `figures/` — list all missing files (do not flag files that exist only in `pics/`, as those are raw sources not meant to be directly included)
- **Dummy/placeholder figures** still present in `figures/` (`dummy_figures/`, `fig1.png` placeholder, etc.)
- `\includegraphics` with **absolute path** (will break on other machines)
  ```
  ❌ \includegraphics{/Users/myname/paper/figures/fig1.pdf}
  ✅ \includegraphics{figures/fig1.pdf}   ← or with \graphicspath{}
  ```
- Figure label missing inside `figure` environment
- Label placed **after** `\caption{}` instead of inside it:
  ```
  ❌ \caption{...}
     \label{fig:x}           ← label after caption: causes wrong hyperlink page
  ✅ \caption{...\label{fig:x}}   ← inside caption
  ✅ \label{fig:x}
     \caption{...}           ← before caption: also acceptable
  ```
- Figures defined but **never referenced** in text
- Missing `[width=\linewidth]` or equivalent size specification in `\includegraphics`

**Tables:**

- Table label missing inside `table` environment
- Table defined but never referenced
- `\hline` used instead of `\toprule` / `\midrule` / `\bottomrule` — flag if venue uses booktabs style
- Inconsistent decimal alignment in numeric columns (use `S` column type from `siunitx`)

---

### CHECK C8 — Hidden Human Errors

**Placeholder and incomplete content:**
- `TODO`, `XXX`, `FIXME`, `TBD`, `???`, `[CITE]`, `[REF]`, `[FILL]` anywhere in `.tex` files
- `\tobeupdated` macro (or similar) still called anywhere
- `\journalVersion{}` calls that contain non-empty content (journal-only content in a conference submission or vice versa)

**Inconsistent naming:**
- Method name with different capitalization (`FooNet` vs `Foonet` vs `FOONET`) across all files
- Dataset name used inconsistently (`KITTI` vs `Kitti` vs `kitti`)
- Metric name used inconsistently (`F1-score` vs `F1 score` vs `F1`)

**Leftover template content:**
- Default template title not replaced
- `Anonymous Authors` still in `\author{}` for non-blind submissions (or real author names present in blind submissions)
- Commented-out sections with old content still in final files
- Content in `deprecated/` folder that is still `\input`-ted from the root `.tex` file

**Formatting artifacts:**
- Double spaces between words (common after find-and-replace operations)
- Non-breaking space `~` missing before `\cite` and `\ref`:
  ```
  ❌ " \cite{x}", " \ref{fig:x}"
  ✅ "~\cite{x}", "~\ref{fig:x}"
  ```
- Hard-coded line breaks `\\` in normal prose (outside tables, equations, and `\myparagraph`)
- `\vspace{-Xmm}` hacks used excessively to meet page limits — list all occurrences and total accumulated space

---

### CHECK C9 — Academic Writing (LaTeX-Detectable)

These issues are checkable by reading the `.tex` source directly.

**Unit formatting:**
```
❌ 10cm, 5Hz, 100ms, 1.5m
✅ 10\,cm, 5\,Hz, 100\,ms, 1.5\,m
```

**Number formatting:**
```
❌ 10000, 1000000
✅ 10,000, 1,000,000
```

**Punctuation patterns:**
```
❌ i.e. the result     →  ✅ i.e., the result
❌ e.g. KITTI          →  ✅ e.g., KITTI
❌ et al               →  ✅ et al.
❌ etc in the text     →  ✅ among others (or list exhaustively)
```

**`state-of-the-art` consistency:**
```
❌ "state of the art method"      →  ✅ "state-of-the-art method" (adjective: hyphenated)
❌ "the state-of-the-art"         →  ✅ "the state of the art" (noun: no hyphen)
```

**Acronym first use:**
- Verify acronyms are expanded on first use in the **abstract** and again on first use in the **body**
- Flag acronyms used before their first expansion
- Flag acronyms re-expanded multiple times unnecessarily

**Sentence-level patterns (grep-detectable):**
- Sentences or paragraphs beginning with `"And "`, `"But "`, `"Or "`
- `"etc."` in formal prose
- `"basically"` or `"Basically"` as a sentence opener
- `"In this section"` or `"In this paper"` as a vague paragraph opener

**Figure reference order:**
- Figure referenced in text before it appears in the document (check paragraph order vs. figure position)
- Equation referenced before it is defined

---

## Output Format

### 1. Critical Errors

Issues that will cause compile failure, wrong output, or submission rejection.

| # | File | Location | Issue | Impact |
|---|------|----------|-------|--------|

### 2. Potential Problems

Issues that will not break compilation but represent real errors.

| # | File | Location | Issue | Impact |
|---|------|----------|-------|--------|

### 3. Style Improvements

Inconsistencies and formatting issues.

| # | File | Location | Issue | Suggestion |
|---|------|----------|-------|------------|

### 4. Infrastructure Suggestions

Recommendations for a safer LaTeX workspace setup.

- ...

---

## Phase 1 Complete — Awaiting User Decision

All issues are listed above with numbers `[1]`, `[2]`, `[3]`...

**Please respond with one of:**

- `fix [numbers]` — e.g., `fix 1, 3, 5` to apply those fixes
- `fix all critical` — to fix only critical errors
- `fix all` — to apply all suggested fixes
- `discard [numbers]` — to skip specific issues

**I will not modify any files until you confirm.**
