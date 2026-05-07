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

## 0.1 MODE SELECTION (read this first)

Real usage is bimodal — pick the mode from the user's first message, don't silently start the wrong workflow.

- **Edit mode (default for Codex / short messages)** — local change to an existing deck. Triggers: short messages (≤ ~80 chars), references to a specific frame ("frame 12", "第 7 页"), relative directional language ("往左一点", "再大一点", "改回去"), color/label/spacing tweaks, or pasted-back earlier versions. Go straight to `tweak` (§2.12). Read **only** the targeted frame, not the whole deck. Use `apply_patch` with the smallest diff that achieves the change.

- **Author mode** — deck does not exist or substantial new content. Triggers: "create slides", "make a presentation", "from this paper", "做一个 X 的 slides". Run the full `create` workflow (Phase 0–5).

If unclear, ask one question — "想做小修改还是从头创作 slides?" — and wait. A long pasted block of LaTeX usually means edit mode (the user is showing you current state), not author mode.

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
11. **Reference slide — duration-conditional.** Required for **talks ≥ 15 min** (`\begin{thebibliography}{9}` with `\small`). For **≤ 10 min conference talks**, a corner attribution suffices (e.g. `Open code: \texttt{github.com/...}` in `\scriptsize\color{gray}`). Ask the user before adding a full references slide on short talks.
12. **Color and contrast standards** — contrast ≥ 4.5:1 (WCAG AA). Never red+green binary. Semantic colors: `\pos{}` = positive (blue), `\con{}` = negative (orange), `\HL{}` = emphasis (green). Palette ≤ 3-5 colors.
    - **Highlight nouns, not phrases** (≤ 4 words; never verbs / clauses).
    - **One role per color across the deck.** If stronger emphasis needed, add a fourth color (`\red{}`, `cbRed = #D62728`) reserved for errors / counter-examples.
    - **Prefer `\textbf{}` over color** for in-slide structural headings.
    - **Max 2 distinct semantic colors active per slide** (plus neutral text).
    - **`\con` is orange (`#DE8F05`), not red.** When the user asks for "red" in an error context, define `cbRed` rather than re-tinting `\con`.
13. **Visual hierarchy in font sizes** — never use `\tiny` for any user-facing content.
14. **Backup slides — duration-conditional.** Required for **≥ 20 min** talks and defenses. For **≤ 15 min**, default to **one targeted backup slide**, not 3-5; for **≤ 10 min**, optional — ask the user which 1-2 questions they expect. Use `\appendix` before backups; backups don't count toward timing.
15. **Columns layout** — use `\begin{columns}[T]` + `\column{W\textwidth}`. Rules:
    - Comparison: two columns at `0.48\textwidth` each (0.04 gap).
    - Figure + text: figure `0.45-0.55`, text `0.40-0.50`, gap `0.05`.
    - Three columns max: each `≤ 0.30\textwidth`, only for short items.
    - Never nest columns. Always `[T]` (top-align).
16. **Parameterize TikZ geometric constants.** For diagrams with **≥ 3 named nodes** or **≥ 4 explicit coordinates**, hoist every layout-affecting number into `\def\name{...}` at the top of `tikzpicture`; use `\pgfmathsetmacro` for derived values. Required parameter set: positions (`\sideX`, `\sideY`, `\nodeR`), sizes (`\nodeRadius`, `\boxWidth`), spacing (`\arrowSep`, `\labelSep`, `\cloudStopX`). **Why**: the user will say "P2 往上一点" or "云缩小" — that should be a one-line `\def` change, not 8 hardcoded coordinates. Apply to every interaction / flow / dependency diagram.

---

## 2. ACTIONS

Parse the user's request to determine which action to run. If no action specified, ask.

### 2.1 `compile [file] [--full]`

**Default: 1-pass incremental.** A single `xelatex -interaction=nonstopmode FILE.tex` is sufficient when no new `\label` / `\cite` / `\ref` were added. Tweak loops want fast feedback.

```bash
xelatex -interaction=nonstopmode FILE.tex
```

**`--full`: 3-pass XeLaTeX + bibtex.** Required when new citations / labels / cross-references / ToC entries were added.

```bash
xelatex -interaction=nonstopmode FILE.tex
bibtex FILE
xelatex -interaction=nonstopmode FILE.tex
xelatex -interaction=nonstopmode FILE.tex
```

**Auto-decide:** if `git diff` of the .tex shows any added line containing `\cite`, `\label`, `\ref`, or `\bibliography`, use `--full`; otherwise default to 1-pass. Always report which mode was used.

Post-compile checks:
- Grep log for `Overfull \\hbox` warnings (count and locations)
- Grep for `Undefined control sequence` or `undefined citations`
- Grep for `Label(s) may have changed` (re-run with `--full` if it appears in 1-pass mode)
- Open PDF for visual verification
- Report: success/failure, overfull count, undefined items, page count, mode used

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

### 2.12 `tweak [file] [frame] [描述]`

**Default action for edit mode.** Surgical, single-frame, minimal-edit modifications. Three rules:

1. **Read-Edit-Verify, not Read-Rewrite-Verify.** Read **only** the targeted frame (from `\begin{frame}` to `\end{frame}`). Use `apply_patch` with the smallest diff — do not regenerate the surrounding frame.
2. **Preserve user-introduced micro-adjustments.** `\vspace{-0.3em}`, `\hspace*{-0.5em}`, manual line breaks, custom-defined colors, commented-out alternatives are previous tuning rounds — load-bearing even when they look redundant.
3. **Compile + visually verify the affected frame, immediately.** 1-pass `compile`, not `--full`. Don't batch multiple tweaks across frames before recompiling.

**Reporting:** after each tweak, in one line, state what changed and what was preserved. Lets the user verify scope without diffing.

**Natural-language interpretation** (中文/English): see `references/tikz-standards.md` → "Natural-Language Adjustment Vocabulary" for conservative defaults (~10-20% per step, not 50%).

**Reversal triggers** ("改回去", "变回 X", "还是 Y 吧", or pasted earlier version): jump to §3.5 — do not "improve" the pasted text.

### 2.13 `imitate [target_frame] [reference.pdf:page]`

Replicate the **layout pattern** of a reference page (typically the user's prior conference slides) onto a new or existing frame. Different from `extract-figures` — that reuses pixels; `imitate` reuses structure.

**Workflow:**
1. Render: `pdftoppm reference.pdf -f PAGE -l PAGE -r 200 /tmp/ref-PAGE.png`, then read the PNG.
2. Identify the layout schema in plain English (column split, standout elements, title strategy, density).
3. Map the user's content to layout slots — state the mapping back before generating code.
4. Reproduce the schema, not the pixels.
5. Show the empty skeleton (`% TODO`) first; get user thumbs-up; then fill.

**Common schemas:**
- *Two-pill summary* — two pills (rounded TikZ box, filled accent) over two panels (rounded box, white fill, itemize). Used for "two prior works" / "two contributions".
- *Keynote-style title page* — `\begin{frame}[plain,noframenumbering]` + manual `\Huge\bfseries` + university logos in a `tabular` row (replaces `\titlepage`).
- *Left-figure-right-text columns* — figure flush with slide edge via `\hspace*{-0.5em}`; text column gets `[T]` top-align.

### 2.14 `poster [slides_file] [size]`

Convert an existing Beamer slides deck (or a paper) into a `beamerposter` poster.

```latex
\documentclass[final]{beamer}
\usepackage[orientation=portrait,size=a1,scale=1.2]{beamerposter}
% reuse the same theme, colors, \pos / \con / \HL macros, TikZ commands
```

Layout: 2 columns (Madrid-style); each `\begin{frame}...\end{frame}` becomes a `\begin{block}{title}...\end{block}` cell flowing down a column. Drop title slide (replace with `\maketitle` header), drop Thank You / References slides — corner attribution suffices.

Sizing: body text `\large` minimum; math standard size (1-3 m viewing distance ≈ projector rules); TikZ figures scale ~1.5× from slide size, verifying arrow widths and font sizes scale too. Compile with `xelatex`. Verify with `pdftoppm -r 100` thumbnail — if it doesn't read at 100px wide, it won't read across a hallway.

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

## 4.5 HANDLING USER REVERSALS

About 1 in 10 user messages is "改回去" / "变回 X" / "还是 Y 吧" / pasting back an earlier version. The user has decided your previous change was wrong; the goal is to undo cleanly without re-introducing earlier issues.

**Protocol:**

1. **The user's pasted text is authoritative.** Don't "polish" or "fix" it on the way back in. Note any concerns at the end — don't silently change them.
2. **Use `git diff` to identify what to revert** — never guess from memory.
3. **Preserve all manual `\vspace{}`, `\hspace{}`, custom colors, and commented-out alternatives** in the user's pasted version. Those are micro-adjustments already paid for.
4. **Confirm scope** before editing: "改回的是 *frame 8* 还是 *整段* 的 styling?" Reversals are usually partial.
5. **Do not re-suggest the rejected change.** If the reverted state has a real problem, report it as a fact and let the user choose.

A reversal means the cost of the previous change exceeded its benefit *for them*. Trust that — fidelity to preference, not optimization.

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

**`\mathbf{X}` looks visibly different from the surrounding math font**
With OTF math fonts loaded via `unicode-math` (`concmath-otf`, `stix2-math`, etc.), `\mathbf` defaults to upright bold in the *text* font. Fix priority: (1) use `\symbf{X}` instead of `\mathbf{X}` — matched bold for the math font; (2) configure explicitly: `\setmathfont{...}[BoldFont={...}, BoldFeatures={...}]`; (3) fallback `\renewcommand{\mathbf}[1]{\boldsymbol{#1}}`. Verify with `$x \mathbf{x} \symbf{x} \boldsymbol{x}$` side-by-side.

**User compiled with `pdflatex` instead of `xelatex` (Hard Rule 5 violation, silent visual regression)**
Add to any custom theme `.sty`:
```latex
\RequirePackage{iftex}
\ifPDFTeX
  \PackageWarningNoLine{mytheme}{This theme is designed for XeLaTeX or LuaLaTeX. pdflatex may render fonts incorrectly.}
\fi
```

**Equations overflow slide width**
Use `\begin{align}` with line breaks → introduce intermediate variables → `\resizebox` as last resort.

**Content visually overflows inside blocks but 0 compiler warnings**
Beamer suppresses overflow warnings inside block environments. Vertical: remove `\vspace{-Xpt}`, limit box to one equation OR few bullets. Horizontal: replace `\qquad` with `\quad`, break equation. Always visually verify every colored box in PDF.
