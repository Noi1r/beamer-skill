---
name: beamer
description: |
  Beamer LaTeX slide workflow: create, compile, review, and polish academic presentations.
  Use this skill whenever the user works on Beamer .tex slide decks, or asks to create slides,
  make a presentation, prepare a lecture, build a talk, or generate Beamer slides from a paper.
  Covers: creation, editing, compilation, proofreading, visual audit, pedagogical review,
  TikZ diagrams, figure extraction, and comprehensive quality checks.
  Trigger on: beamer, slides, lecture, presentation, seminar talk, conference talk, defense slides,
  tikz, compile latex, proofread slides, slide review, 讨论班, 论文讲解.
  Do NOT trigger on: powerpoint, pptx, PPT, 做PPT — use the powerpoint-slides skill instead.
argument-hint: "[action] [file] — actions: create, compile, review, audit, pedagogy, tikz, excellence, devils-advocate, visual-check, validate, extract-figures"
allowed-tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "Agent", "AskUserQuestion", "TaskCreate", "TaskUpdate", "TaskList", "TaskGet"]
---

# Beamer Slide Workflow

Universal skill for academic Beamer presentations. Full lifecycle:
create → compile → review → polish → verify.

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
  - Code listings: `\usepackage{listings}` — set `basicstyle=\ttfamily\small`, `keywordstyle=\bfseries\color{positive}`, `commentstyle=\color{neutral}`. Max 10 lines of code per slide; highlight key lines with `escapeinside={(*@}{@*)}` and `\colorbox`.
  - Pseudocode: `\usepackage{algorithmic}` — simpler than algorithm2e, suitable for short procedures.
- **Data plotting** (add only when needed):
  - `\usepackage{pgfplots}` + `\pgfplotsset{compat=1.18}` — data-driven plots (bar, scatter, error bars, histograms). Far more efficient than manual TikZ coordinate plots for data visualization.
  - `\usepackage{subcaption}` — multiple subfigures in one frame via `\begin{subfigure}{0.48\textwidth}`.
- **Theorem environments**: Beamer provides `theorem`, `lemma`, `corollary`, `definition`, `example`, `proof` out of the box (styled by the theme). Use them for formal statements — they get automatic numbering and consistent styling. Customize with `\setbeamertemplate{theorems}[numbered]` or `[ams style]`.

---

## 1. HARD RULES (Non-Negotiable)

1. **No overlays** — never use `\pause`, `\onslide`, `\only`, `\uncover`. Use multiple slides for progressive builds, color emphasis for attention. **Common replacement patterns:**
   - *Progressive formula build-up*: slide A shows the base equation, slide B copies it and adds the next term (grayed-out terms from slide A stay as context). Copy-paste the full frame and extend.
   - *Step-by-step list reveal*: slide A shows items 1-2, slide B shows 1-4 (items 1-2 repeated). Use `\textcolor{neutral}{}` on previously-shown items to mute them.
   - *Diagram incremental build*: slide A shows the base structure, slide B adds annotations/edges. Duplicate the `tikzpicture` and append new elements — never delete prior elements.
2. **Max 2 colored boxes per slide** — more dilutes emphasis. Demote transitional remarks to plain italic.
3. **Motivation before formalism** — every concept starts with "Why?" before "What?".
4. **Worked example within 2 slides** of every definition.
5. **XeLaTeX only** — never pdflatex.
6. **Beamer .tex is the single source of truth** — TikZ diagrams, content, notation all originate here.
7. **Verify after every task** — compile, check warnings, open PDF.
8. **Telegraphic style** — keyword phrases, not full sentences. Slides are speaker prompts, not manuscripts. Exception: framing sentences that set up a definition or transition. Each bullet item should be ≤2 lines (~15 words) — if longer, split into sub-items or rewrite more concisely.
9. **Every slide earns its place** — each slide must contain at least one substantive element (formula, diagram, table, theorem, or algorithm). A slide with only 3 short bullets and nothing else must be merged or enriched.
10. **Box-interior overflow guard** — `alertblock`, `exampleblock`, and `block` environments add internal padding (~15% less width, ~12-16pt extra height for title bar + vertical padding). Content that fits on a bare slide can overflow inside a box in **both directions**. Rules:
    - **Vertical overflow** (most common, hardest to detect): display math (`\[ \]`) + text below it inside a single box easily exceeds vertical capacity. Limit box content to **one display equation OR 2-3 short bullet items** — not both. Never use aggressive `\vspace{-Xpt}` inside a box; it pulls bottom content past the border.
    - **Horizontal overflow**: never use `\qquad` inside a box; use `\quad` or `,`. If a display equation is wider than ~70% of `\textwidth` on a bare slide, reformat before placing inside a box.
    - **Beamer suppresses overfull warnings inside blocks** — zero compile warnings does NOT guarantee no visual overflow. Always visually verify every box in the PDF.
11. **Reference slide** — the second-to-last slide (before Thank You) must be a **References** slide listing key cited works. Use `\begin{thebibliography}{9}` with `\small`. Include the primary paper and 3-5 most relevant references.
12. **Color and contrast standards** — text-background contrast ratio ≥ 4.5:1 (WCAG AA). Never use red+green for binary contrasts (color blindness affects ~8% of men). Prefer blue+orange. Semantic color commands defined in preamble: `\pos{}` = positive/correct (blue), `\con{}` = negative/limitation (orange), `\HL{}` = emphasis/key finding (green), `\textcolor{neutral}{}` = de-emphasized. These are color-blind safe. Limit total palette to 3-5 colors.
13. **Visual hierarchy in font sizes** — slide title: 20-24pt (beamer default), key findings/theorems: normal size with `\textbf`, supporting text: normal, labels/captions: `\small` minimum. Never use `\tiny` for any user-facing content.
14. **Backup slides** — after the Thank You slide, include 3-5 backup slides for anticipated questions. Use `\appendix` before backup section. Separate from main deck with a `\begin{frame}{Backup Slides}\end{frame}` divider. Backup slides should NOT count toward the timing allocation. **Content to include** (derive from Phase 0 material analysis):
    - Full proof of the main theorem (if only a sketch was shown in the main deck)
    - Parameter choices / security analysis details (for crypto/protocol papers)
    - Extended comparison table with additional baselines
    - Experimental setup details (hardware, hyperparameters, datasets)
    - Definitions of prerequisites that were assumed known (in case someone asks)
15. **Columns layout** — use `\begin{columns}[T]` + `\column{W\textwidth}` for side-by-side content. Rules:
    - Comparison / parallel content: two columns at `0.48\textwidth` each (leave 0.04 gap).
    - Figure + text: figure column `0.45-0.55`, text column `0.40-0.50`, gap `0.05`.
    - Three columns maximum: each `≤ 0.30\textwidth`, only for short items (icons, stats, one-liners).
    - Never nest columns inside columns.
    - Always use `[T]` (top-align) unless deliberately centering.
    - Columns content follows the same density constraints as regular slides.

---

## 2. ACTIONS

Parse `$ARGUMENTS` to determine which action to run. If no action specified, ask.

### 2.1 `compile [file]`

3-pass XeLaTeX + bibtex for full citation resolution.

```bash
# Adapt TEXINPUTS/BIBINPUTS to your project's preamble/bib locations
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

**Collaborative, iterative lecture creation. Strict phase gates — never skip ahead.**

#### Phase 0: Material Analysis (if papers/materials provided)

**Read first, ask later.** Must understand the content before asking meaningful questions.

- Read the full paper/materials thoroughly
- Extract: core contribution, key techniques, main theorems, comparison with prior work
- Map notation conventions
- Identify the paper's logical structure and which parts are slide-worthy
- Internally note: prerequisite knowledge, natural section boundaries, what could be skipped or expanded

**Do NOT present results or ask questions yet — proceed directly to Phase 1.**

#### Phase 1: Needs Interview (MANDATORY — informed by Phase 0)

Conduct a **content-driven interview** via AskUserQuestion. The questions below are the **minimum required set** — you MUST also add paper-specific questions derived from Phase 0.

**Minimum required questions** (always ask):
1. **Duration**: How long is the presentation?
2. **Audience level**: Who are the listeners?

**Optional question** (ask when the talk is a journal club, defense, or user seems to want rehearsal support):
3. **Speaker notes**: Would you like speaker notes in the Presenter View? If yes, `\note{}` blocks will be added per frame with telegraphic talking points (key message, transition cue, anticipated questions). Requires adding `\setbeameroption{show notes on second screen=right}` to the preamble for dual-screen display.

**Content-driven questions** (derive from Phase 0, ask as many as needed):
- **Prerequisite knowledge**: List concrete technical dependencies identified in Phase 0. Ask which ones the audience knows. E.g., "The paper builds on sumcheck and polynomial commitments. Should I review these?" — not "familiar with basic algebra?".
- **Content scope**: Offer the paper's actual components as options. Ask which to emphasize, skip, or briefly mention.
- **Depth vs. breadth**: If the paper has both intuitive overview and detailed constructions, ask which the user prefers.
- **Paper-specific decisions**: E.g., if the paper compares two constructions, ask whether to present both equally or focus on one.

**Guidelines:**
- Options should come from the paper's actual content, not generic templates.
- 3-6 questions total; don't over-ask, don't under-ask.
- If something is obvious from context (e.g., user said "讨论班"), infer rather than ask.

**Slide count heuristic**: ~1 slide per 1.5-2 minutes.

**Timing allocation table:**

| Duration | Total slides | Intro/Motivation | Methods/Background | Core content | Summary/Conclusion |
|----------|-------------|------------------|-------------------|-------------|-------------------|
| 5min (lightning) | 5-7 | 1-2 | 0-1 | 2-3 | 1 |
| 10min (short) | 8-12 | 2 | 1-2 | 4-5 | 1 |
| 15min (conference) | 10-15 | 2-3 | 2-3 | 5-7 | 1-2 |
| 20min (seminar) | 13-18 | 3 | 2-3 | 6-9 | 2 |
| 45min (keynote) | 22-30 | 4-5 | 5-7 | 10-14 | 2-3 |
| 90min (lecture) | 45-60 | 5-6 | 8-12 | 25-35 | 3-4 |

**Talk-type specific tips:**

| Talk type | Key emphasis | Common mistake |
|-----------|-------------|----------------|
| Lightning (5min) | One core message, no background review | Cramming a full talk into 5 minutes |
| Conference (10-20min) | 1-2 key results, fast methods overview | Too much technical detail, no big picture |
| Seminar (45min) | Deep dive OK, but need visual rhythm | Wall-to-wall formulas without examples |
| Defense/Thesis | Demonstrate mastery, systematic coverage | Skipping motivation, rushing results |
| Journal club | Critical analysis, facilitate discussion | Summarizing without evaluating |
| Grant pitch | Significance → feasibility → impact | Too technical, not enough "why it matters" |

**Time distribution principle**: Spend 40-50% of time on core content (results/techniques). Max 3-4 consecutive theory-heavy slides before a worked example or visual break.

#### Phase 2: Structure Plan (GATE — user must approve before drafting)

Produce a **detailed outline**. For each section:
- Section title
- Number of slides allocated
- Key content points per slide (1-2 lines each)
- TikZ diagrams or figures planned (brief description)
- Notation to introduce

**Present the plan to the user.** Ask: structure OK? Expand/shrink/cut anything?

**Do NOT proceed to drafting until user approves.**

#### Phase 3: Draft (iterative, batched)

**This phase is the most critical. Follow all sub-rules strictly.**

##### 3a. Writing Style

- **Telegraphic keywords**, not full sentences. Exception: one framing sentence per slide to set context.
- **Formulas and analysis interleave tightly** — define a quantity, then immediately state its cost/property/implication on the same slide. Never isolate a formula on one slide and its analysis on the next.
- **No conversational hedging** — never write "wait, not exactly", "actually, let me clarify", or similar. If a point needs qualification, state it precisely from the start.
- **Use `\textbf{}` for key terms** on first introduction; use `\pos{...}` for positive properties, `\con{...}` for drawbacks/limitations, `\HL{...}` for key findings (defined in preamble).

##### 3a-2. Opening and Closing Strategies

**Opening slide (pick one strategy):**
- **Surprising statistic** — a counter-intuitive number that challenges assumptions
- **Provocative question** — something the audience cannot immediately answer
- **Real-world failure/problem** — "System X failed because..." → "How do we prevent this?"
- **Visual demonstration** — show the phenomenon before explaining it

The opening should create tension or curiosity that the rest of the talk resolves.

**Closing strategies:**
- **Call-back to opening** — revisit the opening question/problem, now answered (circular narrative)
- **3 key takeaways** — numbered, telegraphic, one slide. The audience remembers the last thing they see.
- **Open question / future direction** — leave the audience thinking, naturally invites Q&A
- **Never end on a bare "Thank You"** — the second-to-last content slide should deliver the lasting impression. The Thank You slide is a courtesy, not the conclusion.

##### 3b. Mathematical Slide Patterns

Each math-heavy slide should follow one of these patterns:

**Definition slide:**
```
[Framing sentence: why this definition matters]
[Formal definition in display math]
[Key properties / immediate consequences as 2-3 bullet items]
```

**Construction/Algorithm slide:**
```
[One-line goal statement]
[Core equation / algorithm steps]
[Complexity analysis: prover cost, verifier cost, soundness]
```

**Comparison slide:**
```
[Side-by-side table: prior work vs this work]
[1-2 lines highlighting the key difference]
```

**Insight/Remark slide (adds value beyond the paper):**
```
[Observation the paper doesn't emphasize, or a comparison with related work]
[Why this matters / what it implies]
```

Every slide must have a clear **takeaway** — the one thing the audience should remember.

**Theorem/Proof slide (for formal mathematical statements):**
```
[Framing sentence: informal statement of the result]
\begin{theorem}[Optional name]
  [Formal statement in display math]
\end{theorem}
[Key implication or "why this matters" as 1-2 bullets]
```
- Proof on the **next** slide (never cramming theorem + proof on one slide).
- For long proofs: show proof sketch (key steps only), put full proof in backup slides.
- Use `\begin{proof}[Proof sketch]` to signal abbreviated proofs.

**Cross-referencing between slides:**
- Label key slides with `\label{slide:construction}` inside the frame.
- Reference from other slides: `see Slide~\ref{slide:construction}` or clickable `\hyperlink{slide:construction}{\beamerbutton{Back to Construction}}`.
- Especially useful for Q&A — quickly jump back to a referenced result.
- In backup slides, always hyperlink back to the main slide that motivates them.

##### 3c. Content Density Constraints

**Upper bounds (per slide):**
- ≤ 7 bullet points or items
- ≤ 2 displayed equations (more → split)
- ≤ 5 new symbols introduced
- ≤ 2 colored boxes (alertblock/exampleblock)

**Lower bounds (per slide):**
- Each slide MUST contain at least one substantive element: a formula, diagram, table, theorem statement, or algorithm
- A slide with only ≤ 3 short text-only bullets is **too sparse** — merge with an adjacent slide or add a formula/diagram
- Pure text-only bullet slides should be ≤ 30% of the total deck

**Density self-check after each batch:**
- Count slides that have zero formulas/diagrams/tables → flag if > 30% of batch
- Count slides with ≤ 3 short items and no math → candidates for merging

##### 3d. Batch Workflow

- Work in batches of 5-10 slides, following the approved structure
- After each batch, **compile** (`xelatex -interaction=nonstopmode`) to catch errors early — fixing 2 issues in a 10-slide batch is far cheaper than fixing 12 issues in a 40-slide deck at Phase 5
- After each batch, self-check: notation consistency, density constraints, motivation-before-formalism
- Continue to next batch only after current batch compiles cleanly and passes self-check

##### 3e. Table Best Practices

- **Always center tables** — wrap every standalone table in `\begin{center}...\end{center}`. Left-aligned tables look misaligned on slides.
- **Never place a table immediately after the frame title** — themes like Madrid have a colored title bar whose bottom edge visually merges with `\toprule`, making the table appear embedded in the title. Always insert `\vspace{4pt}` (or introductory text) between the title and the first `\toprule`. The compiler emits **zero warnings** for this — it can only be caught by visual inspection.
- Always use `booktabs` (`\toprule`, `\midrule`, `\bottomrule`) — never vertical lines (`|`).
- Column alignment: numbers right-aligned (`r`), text left-aligned (`l`), short labels centered (`c`).
- Max 6-7 columns, 8-10 rows per slide. More → split across slides following these pagination rules:
  - Each continuation slide repeats the full header row (`\toprule` + header + `\midrule`).
  - Append " (cont'd)" to the frame title: `\frametitle{Results (cont'd)}`.
  - Split at logical row group boundaries (e.g., between algorithm families, dataset groups) — never mid-group.
  - Last page must have ≥3 data rows; if fewer, merge with the previous page.
  - Each page gets its own takeaway line below the table if the subset tells a different story.
- Use `\resizebox{\textwidth}{!}{...}` only as last resort — prefer reducing columns/rows first.
- Highlight key cells with `\cellcolor{positive!15}` or `\textbf{}` — draw the eye to the result.
- For comparison tables: bold the best result in each row/column.
- Caption below table in `\small`, or use the frame title as the implicit caption.

##### 3f. Algorithm and Code Display

- Pseudocode ≤ 10 lines per slide. If longer, split into "high-level overview" and "key subroutine" slides.
- Highlight the critical line(s): use `escapeinside` in listings or `\colorbox` in algorithm2e.
- Input/output clearly stated at the top of the algorithm.
- Use consistent naming: variables in math italic (`$x$`), functions in sans-serif or typewriter.
- For code (not pseudocode): `\ttfamily\small`, syntax highlighting via listings. Show only the relevant fragment — never dump an entire source file.

#### Phase 4: Figures

- TikZ diagrams in Beamer source (single source of truth)
- Apply TikZ quality standards (see Section 2.6)
- R/Python scripts if needed

**Data visualization guidelines (journal figures ≠ slide figures):**
- **Simplify ruthlessly** — remove minor gridlines, detailed legends (label directly on plot), secondary axes. Show only data supporting the current slide's point.
- **Enlarge everything** — axis labels ≥ 18pt, line width 2-4pt, marker size 8-12pt. If unreadable at 2-3 feet from laptop, it fails on a projector.
- **Direct labeling** — label lines/bars directly instead of using a separate legend. Reduces cognitive load.
- **One message per figure** — split multi-panel journal figures across multiple slides. Each slide shows one comparison.
- **Highlight the result** — key data in bold saturated color, comparison data in muted gray. Use `\textcolor{blue!70!black}{...}` for the key finding, gray for reference.
- **Color-blind safe palette** — prefer blue (`#0173B2`) + orange (`#DE8F05`) over red + green. Add line style differences (solid/dashed/dotted) as redundant encoding. In TikZ:
  ```latex
  \definecolor{cbBlue}{HTML}{0173B2}
  \definecolor{cbOrange}{HTML}{DE8F05}
  \definecolor{cbGreen}{HTML}{029E73}
  \definecolor{cbPurple}{HTML}{CC78BC}
  ```
- **Progressive disclosure** — for complex figures, build incrementally: axes → first dataset → comparison → annotation with finding. Use separate slides (not `\pause`).
- **Subfigures** — use `\begin{subfigure}{0.48\textwidth}` (requires `subcaption` package) for side-by-side panels in one frame. Each subfigure gets its own `\caption{(a) ...}`. Preferred over raw `\includegraphics` side-by-side when panels need individual captions. Max 2 subfigures per slide (4 panels = split across 2 slides).

**pgfplots for data-driven figures (preferred over manual TikZ for data):**

Use `pgfplots` whenever plotting numerical data. It handles axes, legends, scaling, and data loading automatically.

```latex
% Bar chart from inline data
\begin{tikzpicture}
\begin{axis}[
  ybar, bar width=12pt,
  xlabel={Method}, ylabel={Accuracy (\%)},
  symbolic x coords={Baseline, Ours, Oracle},
  xtick=data, ymin=0, ymax=100,
  nodes near coords, every node near coord/.append style={font=\small},
  width=0.85\textwidth, height=5cm
]
  \addplot coordinates {(Baseline,72) (Ours,89) (Oracle,95)};
\end{axis}
\end{tikzpicture}
```

```latex
% Line plot from CSV file
\begin{tikzpicture}
\begin{axis}[
  xlabel={Epoch}, ylabel={Loss},
  legend pos=north east, grid=major,
  width=0.85\textwidth, height=5cm
]
  \addplot table[x=epoch, y=train_loss, col sep=comma] {data/results.csv};
  \addplot table[x=epoch, y=val_loss, col sep=comma] {data/results.csv};
  \legend{Train, Validation}
\end{axis}
\end{tikzpicture}
```

```latex
% Scatter plot with error bars
\begin{axis}[xlabel={$x$}, ylabel={$y$}]
  \addplot+[only marks, error bars/.cd, y dir=both, y explicit]
    coordinates {(1,2)+-(0,0.3) (2,3.5)+-(0,0.5) (3,5.1)+-(0,0.4)};
\end{axis}
```

- Always set explicit `width` and `height` to prevent overflow.
- Use `\addplot table[col sep=comma]{file.csv}` to load data from file — keeps .tex clean.
- For presentation: enlarge tick labels (`tick label style={font=\small}`), thicken lines (`thick`), use semantic colors (`color=positive`).

#### Phase 5: Quality Loop (MANDATORY — iterative)

After completing the full draft, enter the quality loop:

```
┌─→ 5a. Compile (2-pass XeLaTeX)
│   5b. Self-Review (structure + content + visual)
│   5c. Score (apply rubric)
│   5d. Fix all issues found
└── If score < 90 and round < 3: loop back to 5a
    If score ≥ 90 or round = 3: report to user
```

**5a. Compilation**
- 2-pass XeLaTeX compilation (bibtex not needed here — already resolved in Phase 3 batch compiles)
- Check: errors, overfull hbox, undefined references
- Open PDF for visual inspection

**5b. Self-Review** — re-read the .tex and verify:

*Structure:*
- [ ] Slide count matches plan (±2 tolerance)
- [ ] Logical flow: motivation → background → technique → results → summary
- [ ] No section has >4 consecutive formal slides without a worked example or visual break
- [ ] Transition sentences between major sections

*Content density:*
- [ ] No slide has only ≤3 short bullets with no math/diagram
- [ ] Pure text-only slides ≤ 30% of deck
- [ ] No slide exceeds upper bounds (7 bullets, 2 equations, 5 symbols, 2 boxes)

*TikZ and visuals:*
- [ ] No label-label or label-curve overlaps
- [ ] No content overflowing slide boundary
- [ ] **No content overflowing inside colored boxes** (alertblock/exampleblock/block have ~85% effective width; visually verify every box in PDF)
- [ ] **TikZ diagram fits within remaining slide space** — for mixed slides (text + diagram), verify the diagram's bounding box height + text height ≤ ~70mm. Common failure: large TikZ `scale` + multiple equations above = diagram clipped at bottom
- [ ] **All marked points lie on their curves** — for every `\fill`, `\node`, or `\draw` endpoint that should be on a plotted curve, verify the y-coordinate is computed via `\pgfmathsetmacro` from the same function, NOT hardcoded. Visually check in PDF that dots/markers visually sit on the curve line
- [ ] **Dashed reference lines terminate at the curve** — vertical/horizontal guide lines should end exactly where they meet the curve, not at arbitrary y-values
- [ ] Tables fit within slide width
- [ ] All standalone tables are centered (`\begin{center}...\end{center}`)
- [ ] No table's `\toprule` visually merges with the title bar (add `\vspace{4pt}` or text before first table)
- [ ] Font sizes in TikZ ≥ `\footnotesize`
- [ ] Consistent styling across all diagrams

*Notation:*
- [ ] Same symbol used consistently throughout
- [ ] Every symbol defined before use

**5c. Quality Score**

Start at 100. Deduct:

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Compilation failure | -100 |
| Critical | Equation overflow (slide or box-interior) | -20 |
| Critical | TikZ diagram overflows slide boundary (clipped at bottom/right) | -15 per diagram |
| Critical | Undefined control sequence / citation | -15 |
| Critical | Overfull hbox > 10pt | -10 |
| Major | Content overflow inside colored box (vertical or horizontal, visual-only) | -10 per box |
| Major | TikZ marked points not on curve (hardcoded y-values instead of computed) | -8 per diagram |
| Major | Sparse slide (≤3 items, no math/diagram) | -5 per slide |
| Major | TikZ label overlap | -5 |
| Major | Missing references slide | -5 |
| Major | Table not centered (standalone table without `\begin{center}`) | -3 per table |
| Major | Table `\toprule` visually merged with title bar (no spacing after frame title) | -5 per slide |
| Major | Notation inconsistency | -3 |
| Minor | `\vspace` overuse (>3 per slide) | -1 |
| Minor | Font size reduction (`\footnotesize` etc.) | -1 per slide |

**Thresholds:**
- **≥ 90**: Ready to deliver. Report to user.
- **80-89**: Acceptable. Fix remaining majors if possible, report with caveats.
- **< 80**: Must fix. Loop back and resolve critical/major issues.

**5d. Fix** — fix all critical and major issues. Re-compile. If score improves to ≥ 90, exit loop. Max 3 rounds to avoid infinite loops.

#### Post-Creation Checklist (final gate)
```
[ ] Compiles without errors
[ ] No overfull hbox > 10pt
[ ] All citations resolve
[ ] Score ≥ 90
[ ] Every definition has motivation + worked example
[ ] Max 2 colored boxes per slide
[ ] No sparse slides (all slides have substantive content)
[ ] TikZ diagrams visually verified — no overlaps, no overflow, all marked points on curves
[ ] Tables fit within slide boundaries, are centered, and separated from title bar
[ ] No content overflow inside colored boxes (visual PDF check)
[ ] References slide present (second-to-last, before Thank You)
```

### 2.3 `review [file]` or `proofread [file]`

Read-only report, no file edits.

**5 check categories:**

| Category | What to check |
|----------|---------------|
| Grammar | Subject-verb, articles, prepositions, tense consistency |
| Typos | Misspellings, search-replace artifacts, duplicated words, **unreplaced placeholders** (`[name]`, `[TODO]`, `[XXX]`, template remnants) |
| Overflow | Long equations without `\resizebox`, too many items per slide |
| Consistency | Citation format (`\citet`/`\citep`), notation, terminology, box usage, **denominator consistency** across slides |
| Academic quality | Informal abbreviations, missing words, claims without citations, **ambiguous abbreviations** (same abbreviation for different terms) |

**Report format per issue:**
```
### Issue N: [Brief description]
- **Location:** [slide title or line number]
- **Current:** "[exact text]"
- **Proposed:** "[fix]"
- **Category / Severity:** [Category] / [High|Medium|Low]
```

### 2.4 `audit [file]`

Visual layout audit. Read-only report.

**Check dimensions:**
- **Overflow:** Content exceeding boundaries, wide tables/equations
- **Font consistency:** Inline size overrides, inconsistent sizes
- **Box fatigue:** 2+ boxes per slide, wrong box types
- **Spacing:** `\vspace` overuse, structural issues
- **Layout:** Missing transitions, missing framing sentences

**Spacing-first fix principle (priority order):**
1. Reduce vertical spacing (structural changes)
2. Consolidate lists
3. Move displayed equations inline
4. Reduce image/table size with `\resizebox`
5. **Last resort:** `\footnotesize` (never `\tiny`)

### 2.5 `pedagogy [file]`

Holistic pedagogical review. Read-only report.

**13 patterns to validate:**

| # | Pattern | Red flag |
|---|---------|----------|
| 1 | Motivation before formalism | Definition without context |
| 2 | Incremental notation | 5+ new symbols on one slide |
| 3 | Worked example after definition | 2 consecutive definitions, no example |
| 4 | Progressive complexity | Advanced concept before prerequisite |
| 5 | Fragment reveals (problem→solution) | Dense theorem revealed all at once |
| 6 | Standout slides at pivots | Abrupt topic jump, no transition |
| 7 | Two-slide strategy for dense theorems | Complex theorem crammed in 1 slide |
| 8 | Semantic color usage | Binary contrasts in same color |
| 9 | Box hierarchy | Wrong box type for content |
| 10 | Box fatigue | 3+ boxes on one slide |
| 11 | Socratic embedding | Zero questions in entire deck |
| 12 | Visual-first for complex concepts | Notation before visualization |
| 13 | Side-by-side for comparisons | Sequential slides for related definitions |

**Deck-level checks:** Narrative arc, pacing (max 3-4 theory slides before example), visual rhythm (section dividers every 5-8 slides), notation consistency, student prerequisite assumptions.

### 2.6 `tikz [file]`

TikZ diagram review and extraction.

**Quality standards:**
- Labels NEVER overlap with curves, lines, dots, or other labels
- When two labels are near the same vertical position, stagger them
- Visual semantics: solid=observed, dashed=counterfactual, filled=observed, hollow=counterfactual
- Line weights: axes=`thick`, data=`thick`, annotations=`thick` (not `very thick`)
- Standard scale: `[scale=1.1]` for full-width diagrams
- Dot radius: `4pt` for data points
- Minimum 0.2 units between any label and nearest graphical element
- **Mathematical accuracy of plotted points**: NEVER hardcode y-coordinates for points, markers, or dashed-line endpoints that should lie on a plotted curve. ALL such coordinates must be computed from the SAME function used to draw the curve via `\pgfmathsetmacro`. This applies to:
  - Points marked on a **single curve** (e.g., labeled special values like "BiPerm at ℓ=2")
  - Dashed vertical/horizontal lines that terminate at a curve
  - Intersections of **two curves**
  - Any annotation anchored to a curve position

  **Common mistake** (WRONG — hardcoded y that doesn't match the curve):
  ```latex
  \draw[thick] plot[domain=0.8:10] (\x, {0.3*\x + 2.7/\x});
  \draw[dashed] (2, 0) -- (2, 3.2);  % BAD: 3.2 is not 0.3*2+2.7/2=1.95
  \node at (2, 3.2) {BiPerm};         % BAD: label floats above the curve
  ```

  **Correct pattern** (ALWAYS compute from the function):
  ```latex
  \draw[thick] plot[domain=0.8:10] (\x, {0.3*\x + 2.7/\x});
  \pgfmathsetmacro{\yTwo}{0.3*2 + 2.7/2}  % = 1.95, exactly on curve
  \draw[dashed] (2, 0) -- (2, \yTwo);
  \fill (2, \yTwo) circle (2pt);
  \node[above left] at (2, \yTwo) {BiPerm};
  ```

  For curve intersections, solve the system algebraically first, then encode the exact formula:
  ```latex
  % Intersection of y=0.5x and y=2.5/x+0.3:
  % 0.5x = 2.5/x + 0.3 => x^2 - 0.6x - 5 = 0 => x = (0.6+sqrt(20.36))/2
  \pgfmathsetmacro{\xint}{(0.6 + sqrt(20.36))/2}
  \pgfmathsetmacro{\yint}{0.5*\xint}
  \fill (\xint, \yint) circle (3pt);
  ```
- **TikZ diagram sizing on mixed-content slides**: a TikZ diagram sharing a slide with text/equations MUST fit in the remaining vertical space. Beamer 16:9 at 10pt has ~70mm usable height below the title bar. **Before writing the TikZ code**, estimate:
  1. Text + equations above the diagram: count displayed equations (each ~12-15mm) + text lines (~5mm each) + spacing
  2. Remaining height = 70mm − text height
  3. TikZ bounding box height = (max y-coordinate − min y-coordinate) × yscale × 0.3528mm/pt
  4. If the diagram won't fit: reduce `yscale`, shrink coordinate ranges, or move content to a separate slide

  **Safe defaults for mixed slides**: `xscale=0.5-0.7`, `yscale=0.4-0.6`. For full-slide diagrams: `scale=0.9-1.1`.
- **Edge labels on short arrows**: when placing `node[midway, above]` labels on arrows between boxes, the label text can extend past the arrow endpoints into adjacent box borders. Prevention rules:
  1. Estimate label text width vs. arrow length (`right=` gap). If the label is wider than ~80% of the gap, **increase the gap** or **shrink the label font** (`\scriptsize` / `\tiny`).
  2. Use `above=4pt` (or more) instead of bare `above` to add vertical clearance between label and box border.
  3. For flow diagrams with 3+ boxes: total width = (N × box width) + ((N-1) × gap). Must stay ≤ 14cm for 16:9 beamer. Adjust box `text width` and gap together.
  4. When in doubt, compile and visually verify that no label overlaps any box border.

**Extraction to SVG (for web/Quarto use):**
```bash
xelatex -interaction=nonstopmode extract_tikz.tex
PAGES=$(pdfinfo extract_tikz.pdf | grep "Pages:" | awk '{print $2}')
for i in $(seq 1 $PAGES); do
  idx=$(printf "%02d" $((i-1)))
  pdf2svg extract_tikz.pdf tikz_exact_$idx.svg $i
done
```

**TikZ checklist:**
```
[ ] No label-label overlaps
[ ] No label-curve overlaps
[ ] No edge labels overlapping adjacent nodes (check label width vs. arrow length)
[ ] Diagram bounding box fits within remaining slide space (especially mixed text+diagram slides)
[ ] ALL marked points/dots/line-endpoints on curves computed via \pgfmathsetmacro from the SAME function — no hardcoded y-values
[ ] Dashed reference lines terminate exactly at the curve, not at arbitrary coordinates
[ ] Consistent dot style (solid=observed, hollow=counterfactual)
[ ] Consistent line style (solid=observed, dashed=counterfactual)
[ ] Arrow annotations: FROM label TO feature
[ ] Axes extend beyond all data points
[ ] Labels legible at presentation size
[ ] Minimum spacing between labels and graphical elements
```

**Common TikZ diagram patterns:**

Use these as starting points, then customize. All patterns assume `arrows.meta`, `positioning`, `decorations.pathreplacing` libraries are loaded (included in default preamble).

*Flowchart (horizontal):*
```latex
\begin{tikzpicture}[
  box/.style={draw, rounded corners, minimum width=2.2cm, minimum height=0.8cm,
              font=\small, fill=positive!10},
  arr/.style={-{Stealth}, thick}
]
  \node[box] (A) {Step 1};
  \node[box, right=1.5cm of A] (B) {Step 2};
  \node[box, right=1.5cm of B] (C) {Step 3};
  \draw[arr] (A) -- node[above, font=\scriptsize] {label} (B);
  \draw[arr] (B) -- (C);
\end{tikzpicture}
```
- Total width ≤ 14cm for 16:9. Formula: N × box\_width + (N-1) × gap.
- For vertical flows: use `below=` instead of `right=`.

*Timeline:*
```latex
\begin{tikzpicture}
  \draw[-{Stealth}, thick] (0,0) -- (12,0) node[right] {Time};
  \foreach \x/\lab in {1.5/Event A, 5/Event B, 9/Event C} {
    \draw[thick] (\x, 0.15) -- (\x, -0.15);
    \node[above=3pt, font=\small] at (\x, 0.15) {\lab};
  }
\end{tikzpicture}
```

*Tree diagram:*
```latex
\begin{tikzpicture}[
  level distance=1.2cm, sibling distance=2.5cm,
  every node/.style={draw, rounded corners, font=\small, minimum width=1.5cm}
]
  \node {Root}
    child { node {A} child { node {A1} } child { node {A2} } }
    child { node {B} child { node {B1} } };
\end{tikzpicture}
```

*Annotated brace:*
```latex
\draw[decorate, decoration={brace, amplitude=6pt, raise=2pt}]
  (start) -- (end) node[midway, above=10pt, font=\small] {annotation};
```

*Coordinate plot with computed intersection:*
```latex
\begin{tikzpicture}[scale=1.1]
  \draw[-{Stealth}, thick] (0,0) -- (6,0) node[right] {$x$};
  \draw[-{Stealth}, thick] (0,0) -- (0,4) node[above] {$y$};
  \draw[thick, positive] plot[smooth, domain=0.5:5.5] (\x, {0.5*\x});
  \draw[thick, negative, dashed] plot[smooth, domain=0.5:5.5] (\x, {2.5/\x + 0.3});
  % Exact intersection via \pgfmathsetmacro (see quality standards above)
\end{tikzpicture}
```

*Decision diamond (for algorithm flowcharts):*
```latex
\node[diamond, draw, aspect=2, inner sep=1pt, font=\small] (D) {condition?};
\draw[arr] (D) -- node[right, font=\scriptsize] {yes} ++(0,-1.2);
\draw[arr] (D) -- node[above, font=\scriptsize] {no} ++(2.5,0);
```

**Iterative TikZ review loop (for complex diagrams):**

When a TikZ diagram has ≥ 5 nodes or involves plotted curves, run an iterative review:

```
┌─→ Step 1: Mentally render — trace every coordinate, compute where each element appears
│   Step 2: Check for issues — overlaps, misalignments, inconsistent semantics
│   Step 3: Classify — CRITICAL (overlap, wrong semantics, geometric error),
│                       MAJOR (poor spacing, readability), MINOR (aesthetic)
│   Step 4: Fix all CRITICAL and MAJOR issues
│   Step 5: Re-compile and visually verify in PDF
└── If CRITICAL or MAJOR remain and round < 3: loop back to Step 1
    If all clear or round = 3: declare APPROVED or report remaining issues
```

**Verdict criteria:**
- **APPROVED**: Zero CRITICAL, zero MAJOR → diagram is complete
- **NEEDS REVISION**: CRITICAL or MAJOR issues remain → list exact fixes needed
- **REJECTED**: Fundamental structural problems → consider redesigning the diagram

### 2.7 `excellence [file]`

Comprehensive multi-dimensional review. **Use the Agent tool to dispatch review agents in parallel** for maximum efficiency.

**Parallel agent dispatch** — launch these as concurrent Agent calls:

1. **Visual audit agent** — "Read [file]. Check every slide for: overflow, font consistency, box fatigue (2+ boxes), spacing issues, missing transitions. Report per slide with severity. Follow spacing-first fix principle."
2. **Pedagogical review agent** — "Read [file]. Validate 13 pedagogical patterns (motivation before formalism, incremental notation, worked examples, etc.) and deck-level checks (narrative arc, pacing, visual rhythm, notation consistency). Report pattern-by-pattern with status."
3. **Proofreading agent** — "Read [file]. Check grammar, typos, citation consistency (`\citet`/`\citep`), notation consistency, academic quality. Report per issue with location and fix."
4. **TikZ review agent** (if file contains `\begin{tikzpicture}`) — "Read [file]. For every TikZ diagram: check label overlaps, geometric accuracy, visual semantics, spacing. Be merciless — find every flaw. Report per issue with exact coordinates and fixes."
5. **Domain review agent** (optional) — "Read [file]. Verify substantive correctness: assumptions, derivations, citation fidelity."

**After all agents return**, synthesize a combined report:

```markdown
# Slide Excellence Review: [Filename]

## Overall Quality Score: [EXCELLENT / GOOD / NEEDS WORK / POOR]

| Dimension | Critical | Major | Minor |
|-----------|----------|-------|-------|
| Visual/Layout | | | |
| Pedagogical | | | |
| Proofreading | | | |
| TikZ (if any) | | | |

### Critical Issues (Immediate Action Required)
### Major Issues (Next Revision)
### Recommended Next Steps
```

**Quality score rubric** (for multi-agent `excellence` review — complements the numeric score in Phase 5 which is used during `create`):

| Score | Critical issues | Medium issues | Meaning |
|-------|----------------|---------------|---------|
| Excellent | 0-2 | 0-5 | Ready to present (≈ Phase 5 score ≥ 90) |
| Good | 3-5 | 6-15 | Minor refinements (≈ Phase 5 score 80-89) |
| Needs Work | 6-10 | 16-30 | Significant revision (≈ Phase 5 score < 80) |
| Poor | 11+ | 31+ | Major restructuring |

### 2.8 `devils-advocate [file]`

Challenge slide design with 5-7 specific pedagogical questions.

**Challenge categories:**
1. **Ordering** — "Could students understand better if X before Y?"
2. **Prerequisites** — "Do students have the background for this?"
3. **Gaps** — "Should we include an intuitive example here?"
4. **Alternative presentations** — "2 other ways to present this concept"
5. **Notation conflicts** — "This symbol conflicts with earlier usage"
6. **Cognitive load** — "Too many new symbols on this slide"
7. **Standalone readability** — "Does this section stand alone as a book chapter?"

### 2.9 `visual-check [file]`

**PDF→image systematic visual review.** Converts compiled PDF to images, then reviews each slide visually. This catches issues invisible in source code and suppressed by the compiler (especially box-interior overflow).

**Why this matters:** Beamer suppresses overfull warnings inside `block`/`alertblock`/`exampleblock` environments. Zero compile warnings does NOT mean zero visual overflow. This action provides ground truth.

**Workflow:**

1. **Compile** (if not already compiled):
   ```bash
   xelatex -interaction=nonstopmode FILE.tex
   ```

2. **Convert PDF to images** using PyMuPDF:
   ```python
   import fitz
   doc = fitz.open('FILE.pdf')
   n = doc.page_count          # cache before iterating or closing
   zoom = 200 / 72  # 200 DPI
   matrix = fitz.Matrix(zoom, zoom)
   for i in range(n):
       page = doc.load_page(i)
       pixmap = page.get_pixmap(matrix=matrix)
       pixmap.save(f'/tmp/slide-{i+1:03d}.jpg', output='jpeg')
   doc.close()
   print(f'Exported {n} slides')
   ```
   Or via bash: `python3 -c "import fitz; ..."`

   **Fallback** (if PyMuPDF unavailable): Use the Read tool directly on the PDF — Claude Code is multimodal and can read PDF files page by page.

3. **Systematic per-slide inspection** — use Read tool to view each image, checking:
   - [ ] No text overflow at any edge (top, bottom, left, right)
   - [ ] No content overflowing inside colored boxes (check every `alertblock`/`exampleblock`/`block`)
   - [ ] All text legible (no text smaller than readable at presentation distance)
   - [ ] Tables and equations fit within slide width
   - [ ] TikZ labels not overlapping with lines, dots, or other labels
   - [ ] Consistent font sizes across similar slide types
   - [ ] Adequate contrast between text and background
   - [ ] No visual clutter (too many elements competing for attention)

4. **Report** per issue:
   ```
   ### Slide N: [slide title]
   - **Issue:** [description]
   - **Severity:** Critical / Major / Minor
   - **Fix:** [specific recommendation]
   ```

### 2.10 `validate [file] [duration]`

**Automated quantitative validation.** Checks measurable properties without reading content.

**Checks performed:**

1. **Slide count vs. duration** (if duration provided):
   ```bash
   # Get page count
   pdfinfo FILE.pdf | grep "Pages:"
   ```
   Compare against timing allocation table (Section 2.2, Phase 1). Flag if outside recommended range.

2. **Aspect ratio**:
   ```bash
   pdfinfo FILE.pdf | grep "Page size:"
   ```
   Expected: 364.19 x 272.65 pts (16:9 at 10pt) or similar 16:9 ratio. Flag if 4:3 (old projector format).

3. **File size**:
   - \> 50 MB: warning (slow to share/email)
   - \> 100 MB: critical (likely uncompressed images)

4. **Compilation health** (from .log file):
   ```bash
   grep -c "Overfull \\\\hbox" FILE.log
   grep -c "Undefined control sequence" FILE.log
   grep -c "Citation.*undefined" FILE.log
   grep -c "multiply defined" FILE.log
   ```

5. **Source code static checks** (from .tex file):
   - Count `\pause` / `\onslide` / `\only` usage → must be 0 (Hard Rule 1)
   - Count slides with >2 colored boxes → flag violations (Hard Rule 2)
   - Count `\tiny` usage → must be 0 (Hard Rule 13)
   - Check for `\begin{thebibliography}` → warn if missing (Hard Rule 11)

**Report format:**
```
# Validation Report: [Filename]

| Check | Result | Status |
|-------|--------|--------|
| Slide count | N slides / Xmin duration | OK / WARNING |
| Aspect ratio | 16:9 | OK |
| File size | X.X MB | OK / WARNING |
| Overfull hbox | N warnings | OK / CRITICAL |
| Undefined references | N | OK / CRITICAL |
| Overlay commands | N found | OK / VIOLATION |
| Box fatigue violations | N slides | OK / WARNING |
| References slide | Present / Missing | OK / WARNING |

Overall: PASS / PASS WITH WARNINGS / FAIL
```

### 2.11 `extract-figures [pdf] [pages]`

**Extract figures from a paper PDF and prepare them for inclusion in Beamer slides.**

Use when the user wants to reuse figures from an existing paper (their own or a cited work) instead of redrawing in TikZ.

#### Workflow

1. **Identify target figures** — if user doesn't specify pages, use `mcp__pdf-mcp__pdf_get_toc` and `mcp__pdf-mcp__pdf_read_pages` to locate figures in the paper. Ask user which figures to extract if ambiguous.

2. **Extract images** from specified pages:
   ```
   mcp__pdf-mcp__pdf_extract_images(path=PDF_PATH, pages=PAGES, output_dir="figures")
   ```
   Uses `output_dir` to save images directly to `figures/` as PNG files (original resolution, zero token cost).
   Returns `{page, index, width, height, format, file_path}` metadata only — no base64 data.

3. **Rename to descriptive names** following the naming convention:
   ```bash
   mv figures/page3_img0.png figures/fig-LABEL.png
   ```
   Naming convention: `fig-<descriptive-label>.png` (e.g., `fig-architecture.png`, `fig-results-table.png`).

4. **Generate LaTeX snippet** — output ready-to-paste code:

   *Full-width figure:*
   ```latex
   \begin{frame}{Frame Title}
     \begin{center}
       \includegraphics[width=0.85\textwidth]{figures/fig-LABEL.png}
     \end{center}
     \vspace{-6pt}
     {\small Source: \citet{author2024paper}}
   \end{frame}
   ```

   *Figure + text side-by-side (columns layout):*
   ```latex
   \begin{frame}{Frame Title}
     \begin{columns}[T]
       \column{0.50\textwidth}
         \includegraphics[width=\textwidth]{figures/fig-LABEL.png}
         {\small Source: \citet{author2024paper}}
       \column{0.45\textwidth}
         Key observations:
         \begin{itemize}
           \item Point 1
           \item Point 2
         \end{itemize}
     \end{columns}
   \end{frame}
   ```

   *Two figures side-by-side (subfigure):*
   ```latex
   \begin{frame}{Frame Title}
     \begin{figure}
       \begin{subfigure}{0.48\textwidth}
         \includegraphics[width=\textwidth]{figures/fig-LEFT.png}
         \caption{(a) Description}
       \end{subfigure}\hfill
       \begin{subfigure}{0.48\textwidth}
         \includegraphics[width=\textwidth]{figures/fig-RIGHT.png}
         \caption{(b) Description}
       \end{subfigure}
     \end{figure}
   \end{frame}
   ```

5. **Cropping guidance** — if the extracted image contains surrounding text or margins that should be removed, use the `trim` and `clip` options:
   ```latex
   \includegraphics[width=0.85\textwidth, trim=LEFT BOTTOM RIGHT TOP, clip]{figures/fig-LABEL.png}
   ```
   - Units are in `bp` (big points). Estimate from the image dimensions returned by `pdf_extract_images`.
   - Common case: `trim=50 200 50 100, clip` to remove page margins and surrounding text.
   - **Always visually verify** after compilation — cropping coordinates are estimates.

#### Rules

- **Always attribute** — include `{\small Source: ...}` below every extracted figure unless it's the user's own paper.
- **Prefer vector** — if only a single clean figure is on a page, extraction quality is usually sufficient. For complex multi-figure pages, consider redrawing the key figure in TikZ for better quality and consistency.
- **Resolution check** — if extracted image width < 800px, warn the user that it may appear pixelated on a projector. Suggest either finding a higher-resolution source or redrawing in TikZ.
- **Don't extract tables** — tables from PDF rasterize poorly. Always recreate tables in LaTeX using `booktabs`.
- **Preamble dependency** — ensure `\usepackage{graphicx}` is in the preamble (add if missing). If using subfigures, also ensure `\usepackage{subcaption}`.

---

## 3. VERIFICATION PROTOCOL

**Every task ends with verification.** Non-negotiable.

```
[ ] Compiled without errors (xelatex exit code 0)
[ ] No overfull hbox > 10pt
[ ] All citations resolve
[ ] PDF opens and renders correctly
[ ] Visual spot-check of modified slides
```

---

## 4. DOMAIN REVIEW (Template)

For substantive correctness (not presentation), review through 5 lenses:

1. **Assumption stress test** — all assumptions stated, sufficient, necessary?
2. **Derivation verification** — each step follows? Decomposition sums to whole?
3. **Citation fidelity** — slide accurately represents cited paper?
4. **Code-theory alignment** — code implements exact formula from slides?
5. **Backward logic check** — read conclusion→setup, every claim supported?

Severity: CRITICAL = math wrong. MAJOR = missing assumption. MINOR = could be clearer.

---

## 5. TROUBLESHOOTING

**Error:** `! Undefined control sequence. \llbracket`
**Cause:** Missing `stmaryrd` package for double brackets.
**Fix:** Add `\usepackage{stmaryrd}` to preamble. Or define `\newcommand{\llbracket}{[\![}` as fallback.

**Error:** `Overfull \vbox` on a specific slide
**Cause:** Too much content for the frame height.
**Fix priority:**
1. Reduce `\vspace` values
2. Shorten text (telegraphic style)
3. Split into two slides
4. Use `\small` on one element (not the whole slide)
5. Last resort: `\footnotesize` (never `\tiny`)

**Error:** `Font "XXX" not found` with XeLaTeX
**Cause:** System font not installed, or wrong font name.
**Fix:** Use `fc-list | grep "FontName"` to check available fonts. Fall back to default Latin Modern if custom font unavailable.

**Error:** Equations overflow slide width
**Cause:** Long multi-term equation.
**Fix priority:**
1. Use `\begin{align}` with line breaks at natural points (`=`, `+`)
2. Introduce intermediate variables to shorten expressions
3. Use `\resizebox{\textwidth}{!}{...}` as last resort (degrades readability)

**Error:** PDF images don't render in Quarto/web
**Cause:** Browsers cannot display PDF inline.
**Fix:** Convert to SVG: `pdf2svg input.pdf output.svg`. Never use PNG for diagrams (raster = blurry).

**Error:** Content visually overflows inside `alertblock`/`exampleblock`/`block` but compiler reports 0 warnings
**Cause:** Beamer suppresses overflow warnings inside block environments. Two distinct failure modes:
- *Vertical*: display math + text below it exceeds the box's vertical capacity; bottom content spills past the lower border. Aggravated by `\vspace{-Xpt}`.
- *Horizontal*: the block's internal padding reduces effective width by ~15%, so wide equations overflow sideways.
**Fix priority (vertical — most common):**
1. Remove or reduce `\vspace{-Xpt}` inside the box
2. Keep box content minimal: one display equation OR a few bullet items, not both
3. Move explanatory text below the equation to outside the box (plain text after `\end{...block}`)
4. Split into two separate boxes or slides
**Fix priority (horizontal):**
1. Replace `\qquad` with `\quad` or `,`
2. Move secondary notation (e.g., sampling arrows) to a separate line
3. Break the equation with `\\` at natural points
4. Move the equation outside the box
**Never rely on compile warnings alone** — always visually verify every colored box in the PDF.
