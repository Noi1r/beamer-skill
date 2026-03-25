# Beamer Slide Workflow

Universal skill for academic Beamer presentations. Full lifecycle:
**create → compile → review → polish → verify.**

> This file is for OpenAI Codex CLI. For Claude Code, use `SKILL.md` instead.
> Detailed rules for each section are in the `references/` subdirectory — read them when executing the corresponding action.

---

## 0. REFERENCE PREAMBLE

When creating new slides, use this as the default preamble unless the user has a custom template.

```latex
\documentclass[aspectratio=169,10pt]{beamer}

\usetheme{Madrid}
\usecolortheme{default}
\setbeamertemplate{navigation symbols}{}
\setbeamertemplate{footline}[frame number]

\usepackage{amsmath,amssymb,amsthm,booktabs,mathtools}
\usepackage{stmaryrd}  % for \llbracket, \rrbracket
\usepackage{graphicx}  % for \includegraphics (extracted figures)
\usepackage{hyperref}
\usepackage{tikz}
\usetikzlibrary{arrows.meta,positioning,decorations.pathreplacing}

% Semantic colors — use in BOTH text and TikZ for global consistency
\definecolor{positive}{HTML}{0173B2}       % blue (correct, advantage)
\definecolor{negative}{HTML}{DE8F05}       % orange (limitation, drawback)
\definecolor{emphasis}{HTML}{029E73}        % green (highlight, key finding)
\definecolor{neutral}{gray}{0.55}          % muted context
\definecolor{cbPurple}{HTML}{CC78BC}        % additional accent
\newcommand{\pos}[1]{\textcolor{positive}{#1}}
\newcommand{\con}[1]{\textcolor{negative}{#1}}
\newcommand{\HL}[1]{\textcolor{emphasis}{#1}}
```

**Rules:**
- **Always `10pt`** — `11pt` or `12pt` produces oversized, sparse slides.
- **Always `aspectratio=169`** — modern projectors are 16:9.
- **Default presenter**: `\author{Presenter: [name]}` and `\institute{Shanghai Jiao Tong University}`. Override only if user specifies otherwise.
- If user provides a custom preamble, header file, or theme: use theirs.
- Add domain-specific macros (e.g. `\newcommand{\F}{\mathbb{F}}`) as needed.
- **Algorithm/code packages** (add only when needed):
  - Algorithms: `\usepackage[ruled,lined]{algorithm2e}` — set `\SetAlgoLined`, `\DontPrintSemicolon`
  - Code listings: `\usepackage{listings}` — set `basicstyle=\ttfamily\small`, max 10 lines per slide
  - Pseudocode: `\usepackage{algorithmic}` — simpler than algorithm2e, suitable for short procedures
- **Data plotting** (add only when needed):
  - `\usepackage{pgfplots}` + `\pgfplotsset{compat=1.18}` — data-driven plots
  - `\usepackage{subcaption}` — multiple subfigures via `\begin{subfigure}{0.48\textwidth}`
- **Theorem environments**: Beamer provides `theorem`, `lemma`, `corollary`, `definition`, `example`, `proof` out of the box.

---

## 1. HARD RULES (Non-Negotiable)

1. **No overlays** — never use `\pause`, `\onslide`, `\only`, `\uncover`. Use multiple slides for progressive builds, color emphasis for attention.
2. **Max 2 colored boxes per slide** — more dilutes emphasis.
3. **Motivation before formalism** — every concept starts with "Why?" before "What?".
4. **Worked example within 2 slides** of every definition.
5. **XeLaTeX only** — never pdflatex.
6. **Beamer .tex is the single source of truth** — TikZ diagrams, content, notation all originate here.
7. **Verify after every task** — compile, check warnings, open PDF.
8. **Telegraphic style** — keyword phrases, not full sentences. Slides are speaker prompts, not manuscripts.
9. **Every slide earns its place** — each slide must contain at least one substantive element (formula, diagram, table, theorem, or algorithm). A slide with only 3 short bullets must be merged or enriched.
10. **Box-interior overflow guard** — `alertblock`, `exampleblock`, and `block` add internal padding (~15% less width, ~12-16pt extra height). Content that fits on a bare slide can overflow inside a box. Rules:
    - **Vertical overflow**: limit box content to **one display equation OR 2-3 short bullet items** — not both.
    - **Horizontal overflow**: never use `\qquad` inside a box; use `\quad` or `,`.
    - **Beamer suppresses overfull warnings inside blocks** — always visually verify every box in PDF.
11. **Reference slide** — second-to-last slide (before Thank You) must be a **References** slide. Use `\begin{thebibliography}{9}` with `\small`.
12. **Color and contrast standards** — text-background contrast ≥ 4.5:1 (WCAG AA). Never red+green binary contrasts. Semantic colors: `\pos{}` = positive (blue), `\con{}` = negative (orange), `\HL{}` = emphasis (green). Limit palette to 3-5 colors.
13. **Visual hierarchy in font sizes** — never use `\tiny` for any user-facing content.
14. **Backup slides** — after Thank You, include 3-5 backup slides for anticipated questions. Use `\appendix` before backup section. Backup slides do NOT count toward timing.
15. **Columns layout** — use `\begin{columns}[T]` + `\column{W\textwidth}`. Rules:
    - Comparison: two columns at `0.48\textwidth` each (0.04 gap).
    - Figure + text: figure `0.45-0.55`, text `0.40-0.50`, gap `0.05`.
    - Three columns max: each `≤ 0.30\textwidth`, only for short items.
    - Never nest columns. Always `[T]` (top-align).

---

## 2. ACTIONS

Parse the user's request to determine which action to run. If no action specified, ask.

### 2.1 `compile [file]`

3-pass XeLaTeX + bibtex for full citation resolution.

```bash
xelatex -interaction=nonstopmode FILE.tex
bibtex FILE
xelatex -interaction=nonstopmode FILE.tex
xelatex -interaction=nonstopmode FILE.tex
```

Post-compile checks:
- Grep log for `Overfull \\hbox` warnings (count and locations)
- Grep for `Undefined control sequence` or `undefined citations`
- Grep for `Label(s) may have changed`
- Open PDF for visual verification
- Report: success/failure, overfull count, undefined items, page count

### 2.2 `create [topic]`

Collaborative, iterative lecture creation with strict phase gates.

**Phases:** Material Analysis → Needs Interview → Structure Plan → Draft → Figures → Quality Loop

See `references/create-workflow.md` for the complete Phase 0-5 workflow, including:
- Phase 0: Material analysis (read papers, extract key content)
- Phase 1: Needs interview (duration, audience, content scope)
- Phase 2: Structure plan (detailed outline, user approval gate)
- Phase 3: Draft (writing style, math patterns, density constraints, batch workflow)
- Phase 4: Figures (TikZ, data visualization, pgfplots)
- Phase 5: Quality loop (compile → self-review → score → fix, iterative)

**Key constraints during creation:**
- Slide count heuristic: ~1 slide per 1.5-2 min
- Content density: ≤ 7 bullets, ≤ 2 equations, ≤ 5 new symbols, ≤ 2 colored boxes per slide
- Lower bounds: each slide needs at least one substantive element; pure text-only ≤ 30% of deck
- Quality score starts at 100, deduct per issue. Target ≥ 90 to deliver.

### 2.3 `review [file]` or `proofread [file]`

Read-only proofreading report, no file edits. See `references/review-actions.md` for details.

**5 check categories:** Grammar, Typos, Overflow, Consistency, Academic quality.

**Report format per issue:**
```
### Issue N: [Brief description]
- **Location:** [slide title or line number]
- **Current:** "[exact text]"
- **Proposed:** "[fix]"
- **Category / Severity:** [Category] / [High|Medium|Low]
```

### 2.4 `audit [file]`

Visual layout audit. Read-only report. See `references/review-actions.md` for details.

**Check dimensions:** Overflow, Font consistency, Box fatigue, Spacing, Layout.

**Spacing-first fix principle (priority order):**
1. Reduce vertical spacing
2. Consolidate lists
3. Move displayed equations inline
4. Reduce image/table size with `\resizebox`
5. **Last resort:** `\footnotesize` (never `\tiny`)

### 2.5 `pedagogy [file]`

Holistic pedagogical review with 13 validation patterns. Read-only report. See `references/review-actions.md` for details.

**13 patterns:** Motivation before formalism, Incremental notation, Worked examples, Progressive complexity, Fragment reveals, Standout slides, Two-slide theorem strategy, Semantic colors, Box hierarchy, Box fatigue, Socratic embedding, Visual-first, Side-by-side comparisons.

### 2.6 `tikz [file]`

TikZ diagram review and SVG extraction. See `references/tikz-standards.md` for quality standards and common patterns.

**Key rules:**
- Labels NEVER overlap with curves, lines, dots, or other labels
- ALL marked points on curves computed via `\pgfmathsetmacro` — no hardcoded y-values
- Visual semantics: solid=observed, dashed=counterfactual, filled=observed, hollow=counterfactual
- Standard scale: `[scale=1.1]` for full-width, safe defaults for mixed slides: `xscale=0.5-0.7`, `yscale=0.4-0.6`

### 2.7 `excellence [file]`

Comprehensive multi-dimensional review. See `references/review-actions.md` for details.

Dispatch parallel review tasks covering:
1. **Visual audit** — overflow, font consistency, box fatigue, spacing, transitions
2. **Pedagogical review** — 13 patterns + deck-level checks
3. **Proofreading** — grammar, typos, citations, notation, academic quality
4. **TikZ review** (if applicable) — label overlaps, geometric accuracy, visual semantics
5. **Domain review** (optional) — substantive correctness

Synthesize a combined report with quality score: EXCELLENT / GOOD / NEEDS WORK / POOR.

### 2.8 `devils-advocate [file]`

Challenge slide design with 5-7 specific pedagogical questions across categories: Ordering, Prerequisites, Gaps, Alternative presentations, Notation conflicts, Cognitive load, Standalone readability.

### 2.9 `visual-check [file]`

PDF-based visual verification. Convert compiled PDF to images, then inspect each slide for: text overflow, box-interior overflow, legibility, table/equation fit, TikZ label overlaps, font consistency, contrast, visual clutter.

### 2.10 `validate [file] [duration]`

Automated quantitative validation. See `references/review-actions.md` for details.

Checks: slide count vs. duration, aspect ratio, file size, compilation health (overfull hbox, undefined refs), source code static checks (no overlays, no `\tiny`, references slide present).

### 2.11 `extract-figures [pdf] [pages]`

Extract figures from paper PDFs for direct inclusion in slides. Uses PDF reading tools to identify and extract images, saves to `figures/` directory, generates ready-to-paste LaTeX snippets with proper attribution and cropping guidance.

---

## 3. QUALITY SCORING

Start at 100. Deduct per issue:

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Compilation failure | -100 |
| Critical | Equation overflow (slide or box-interior) | -20 |
| Critical | TikZ diagram overflows slide boundary | -15 per diagram |
| Critical | Undefined control sequence / citation | -15 |
| Critical | Overfull hbox > 10pt | -10 |
| Major | Content overflow inside colored box (visual-only) | -10 per box |
| Major | TikZ marked points not on curve (hardcoded y-values) | -8 per diagram |
| Major | Sparse slide (≤3 items, no math/diagram) | -5 per slide |
| Major | TikZ label overlap | -5 |
| Major | Missing references slide | -5 |
| Major | Table `\toprule` visually merged with title bar (no spacing after frame title) | -5 per slide |
| Major | Table not centered (standalone table without `\begin{center}`) | -3 per table |
| Major | Notation inconsistency | -3 |
| Minor | `\vspace` overuse (>3 per slide) | -1 |
| Minor | Font size reduction (`\footnotesize` etc.) | -1 per slide |

**Thresholds:** ≥ 90 Ready | 80-89 Acceptable | < 80 Must fix

**Excellence review rubric (multi-dimensional):**

| Score | Critical | Medium | Meaning |
|-------|----------|--------|---------|
| Excellent | 0-2 | 0-5 | Ready to present |
| Good | 3-5 | 6-15 | Minor refinements |
| Needs Work | 6-10 | 16-30 | Significant revision |
| Poor | 11+ | 31+ | Major restructuring |

---

## 4. VERIFICATION PROTOCOL

**Every task ends with verification.** Non-negotiable.

```
[ ] Compiled without errors (xelatex exit code 0)
[ ] No overfull hbox > 10pt
[ ] All citations resolve
[ ] PDF opens and renders correctly
[ ] Visual spot-check of modified slides
```

---

## 5. DOMAIN REVIEW (Template)

For substantive correctness, review through 5 lenses:

1. **Assumption stress test** — all assumptions stated, sufficient, necessary?
2. **Derivation verification** — each step follows? Decomposition sums to whole?
3. **Citation fidelity** — slide accurately represents cited paper?
4. **Code-theory alignment** — code implements exact formula from slides?
5. **Backward logic check** — read conclusion→setup, every claim supported?

Severity: CRITICAL = math wrong. MAJOR = missing assumption. MINOR = could be clearer.

---

## 6. TROUBLESHOOTING

**`! Undefined control sequence. \llbracket`**
Add `\usepackage{stmaryrd}` to preamble. Or define `\newcommand{\llbracket}{[\![}` as fallback.

**`Overfull \vbox` on a specific slide**
Fix priority: reduce `\vspace` → shorten text → split slide → `\small` on one element → `\footnotesize` (last resort, never `\tiny`).

**`Font "XXX" not found` with XeLaTeX**
Use `fc-list | grep "FontName"` to check. Fall back to Latin Modern.

**Equations overflow slide width**
Use `\begin{align}` with line breaks → introduce intermediate variables → `\resizebox` as last resort.

**Content visually overflows inside blocks but 0 compiler warnings**
Beamer suppresses overflow warnings inside block environments. Vertical: remove `\vspace{-Xpt}`, limit box to one equation OR few bullets. Horizontal: replace `\qquad` with `\quad`, break equation. Always visually verify every colored box in PDF.
