# Create Workflow — Phase 0-5 Detailed Rules

This document contains the complete `create [topic]` workflow for the Beamer skill.

---

## Phase 0: Material Analysis (if papers/materials provided)

**Read first, ask later.** Must understand the content before asking meaningful questions.

- Read the full paper/materials thoroughly
- Extract: core contribution, key techniques, main theorems, comparison with prior work
- Map notation conventions
- Identify the paper's logical structure and which parts are slide-worthy
- Internally note: prerequisite knowledge, natural section boundaries, what could be skipped or expanded

**Do NOT present results or ask questions yet — proceed directly to Phase 1.**

---

## Phase 1: Needs Interview (MANDATORY — informed by Phase 0)

Conduct a content-driven interview. The questions below are the **minimum required set** — you MUST also add paper-specific questions derived from Phase 0.

**Minimum required questions** (always ask):
1. **Duration**: How long is the presentation?
2. **Audience level**: Who are the listeners?

**Optional question** (ask when the talk is a journal club, defense, or user seems to want rehearsal support):
3. **Speaker notes**: Would you like speaker notes in Presenter View? If yes, `\note{}` blocks will be added per frame with telegraphic talking points. Requires `\setbeameroption{show notes on second screen=right}` in preamble.

**Content-driven questions** (derive from Phase 0, ask as many as needed):
- **Prerequisite knowledge**: List concrete technical dependencies. E.g., "The paper builds on sumcheck and polynomial commitments. Should I review these?"
- **Content scope**: Offer the paper's actual components as options. Ask which to emphasize, skip, or briefly mention.
- **Depth vs. breadth**: If the paper has both intuitive overview and detailed constructions, ask which the user prefers.
- **Paper-specific decisions**: E.g., if the paper compares two constructions, ask whether to present both equally or focus on one.

**Guidelines:**
- Options should come from the paper's actual content, not generic templates.
- 3-6 questions total; don't over-ask, don't under-ask.
- If something is obvious from context, infer rather than ask.

**Slide count heuristic**: ~1 slide per 1.5-2 minutes.

**Timing allocation table:**

| Duration | Total slides | Intro/Motivation | Methods/Background | Core content | Summary |
|----------|-------------|------------------|-------------------|-------------|---------|
| 5min (lightning) | 5-7 | 1-2 | 0-1 | 2-3 | 1 |
| 10min (short) | 8-12 | 2 | 1-2 | 4-5 | 1 |
| 15min (conference) | 10-15 | 2-3 | 2-3 | 5-7 | 1-2 |
| 20min (seminar) | 13-18 | 3 | 2-3 | 6-9 | 2 |
| 45min (keynote) | 22-30 | 4-5 | 5-7 | 10-14 | 2-3 |
| 90min (lecture) | 45-60 | 5-6 | 8-12 | 25-35 | 3-4 |

**Talk-type tips:**

| Talk type | Key emphasis | Common mistake |
|-----------|-------------|----------------|
| Lightning (5min) | One core message, no background review | Cramming a full talk into 5 minutes |
| Conference (10-20min) | 1-2 key results, fast methods overview | Too much technical detail, no big picture |
| Seminar (45min) | Deep dive OK, but need visual rhythm | Wall-to-wall formulas without examples |
| Defense/Thesis | Demonstrate mastery, systematic coverage | Skipping motivation, rushing results |
| Journal club | Critical analysis, facilitate discussion | Summarizing without evaluating |
| Grant pitch | Significance → feasibility → impact | Too technical, not enough "why it matters" |

**Time distribution**: 40-50% on core content. Max 3-4 consecutive theory-heavy slides before a worked example or visual break.

---

## Phase 2: Structure Plan (GATE — user must approve before drafting)

Produce a **detailed outline**. For each section:
- Section title
- Number of slides allocated
- Key content points per slide (1-2 lines each)
- TikZ diagrams or figures planned (brief description)
- Notation to introduce

**Present the plan to the user.** Ask: structure OK? Expand/shrink/cut anything?

**Do NOT proceed to drafting until user approves.**

---

## Phase 3: Draft (iterative, batched)

### 3a. Writing Style

- **Telegraphic keywords**, not full sentences. Exception: one framing sentence per slide to set context.
- **Formulas and analysis interleave tightly** — define a quantity, then immediately state its cost/property/implication on the same slide.
- **No conversational hedging** — never write "wait, not exactly", "actually, let me clarify".
- **Use `\textbf{}` for key terms** on first introduction; use `\pos{...}` for positive, `\con{...}` for drawbacks, `\HL{...}` for key findings.

### 3a-2. Opening and Closing Strategies

**Opening slide (pick one strategy):**
- **Surprising statistic** — a counter-intuitive number
- **Provocative question** — something the audience cannot immediately answer
- **Real-world failure/problem** — "System X failed because..."
- **Visual demonstration** — show the phenomenon before explaining it

**Closing strategies:**
- **Call-back to opening** — revisit the opening question, now answered
- **3 key takeaways** — numbered, telegraphic, one slide
- **Open question / future direction** — invites Q&A
- **Never end on a bare "Thank You"** — the second-to-last content slide delivers the lasting impression

### 3b. Mathematical Slide Patterns

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

**Insight/Remark slide:**
```
[Observation the paper doesn't emphasize, or comparison with related work]
[Why this matters / what it implies]
```

**Theorem/Proof slide:**
```
[Framing sentence: informal statement of the result]
\begin{theorem}[Optional name]
  [Formal statement]
\end{theorem}
[Key implication as 1-2 bullets]
```
- Proof on the **next** slide (never cramming theorem + proof on one slide).
- For long proofs: show proof sketch only, full proof in backup slides.

**Cross-referencing:** Use `\label{slide:construction}` and `\hyperlink{slide:construction}{\beamerbutton{Back}}` for navigation, especially in backup slides.

### 3c. Content Density Constraints

**Upper bounds (per slide):**
- ≤ 7 bullet points
- ≤ 2 displayed equations
- ≤ 5 new symbols introduced
- ≤ 2 colored boxes

**Lower bounds (per slide):**
- Each slide MUST contain at least one substantive element
- A slide with only ≤ 3 short text-only bullets is too sparse — merge or enrich
- Pure text-only bullet slides ≤ 30% of total deck

**Density self-check after each batch:**
- Count slides with zero formulas/diagrams/tables → flag if > 30%
- Count slides with ≤ 3 short items and no math → candidates for merging

### 3d. Batch Workflow

- Work in batches of 5-10 slides, following the approved structure
- After each batch, **compile** (`xelatex -interaction=nonstopmode`) to catch errors early — fixing 2 issues in a 10-slide batch is far cheaper than fixing 12 issues in a 40-slide deck at Phase 5
- After each batch: self-check notation consistency, density constraints, motivation-before-formalism
- Continue to next batch only after current batch compiles cleanly and passes self-check

### 3e. Table Best Practices

- **Always center tables** — wrap every standalone table in `\begin{center}...\end{center}`. Left-aligned tables look misaligned on slides.
- **Never place a table immediately after the frame title** — themes like Madrid have a colored title bar whose bottom edge visually merges with `\toprule`, making the table appear embedded in the title. Always insert `\vspace{4pt}` (or introductory text) between the title and the first `\toprule`. The compiler emits **zero warnings** for this — it can only be caught by visual inspection.
- Always `booktabs` (`\toprule`, `\midrule`, `\bottomrule`) — never vertical lines.
- Numbers right-aligned, text left-aligned, short labels centered.
- Max 6-7 columns, 8-10 rows per slide. More → split across slides following these pagination rules:
  - Each continuation slide repeats the full header row (`\toprule` + header + `\midrule`).
  - Append " (cont'd)" to the frame title: `\frametitle{Results (cont'd)}`.
  - Split at logical row group boundaries — never mid-group.
  - Last page must have ≥3 data rows; if fewer, merge with the previous page.
  - Each page gets its own takeaway line below the table if the subset tells a different story.
- Use `\resizebox{\textwidth}{!}{...}` only as last resort — prefer reducing columns/rows first.
- Highlight key cells with `\cellcolor{positive!15}` or `\textbf{}`.
- For comparison tables: bold the best result in each row/column.

### 3f. Algorithm and Code Display

- Pseudocode ≤ 10 lines per slide.
- Highlight the critical line(s) via `escapeinside` or `\colorbox`.
- Input/output clearly stated at top.
- For code (not pseudocode): `\ttfamily\small`, syntax highlighting via listings.

---

## Phase 4: Figures

- TikZ diagrams in Beamer source (single source of truth)
- Apply TikZ quality standards (see `references/tikz-standards.md`)

**Data visualization guidelines (journal figures ≠ slide figures):**
- **Simplify ruthlessly** — remove minor gridlines, detailed legends. Show only supporting data.
- **Enlarge everything** — axis labels ≥ 18pt, line width 2-4pt, marker size 8-12pt.
- **Direct labeling** — label lines/bars directly instead of separate legend.
- **One message per figure** — split multi-panel journal figures across slides.
- **Highlight the result** — key data in bold saturated color, comparison in muted gray.
- **Color-blind safe palette** — blue+orange over red+green. Add line style differences.
- **Progressive disclosure** — build complex figures incrementally across separate slides.
- **Subfigures** — use `\begin{subfigure}{0.48\textwidth}` for side-by-side panels. Max 2 per slide.

**pgfplots for data-driven figures:**

```latex
% Bar chart
\begin{axis}[ybar, bar width=12pt, xlabel={Method}, ylabel={Accuracy (\%)},
  symbolic x coords={Baseline, Ours, Oracle}, xtick=data, nodes near coords]
  \addplot coordinates {(Baseline,72) (Ours,89) (Oracle,95)};
\end{axis}
```

```latex
% Line plot from CSV
\begin{axis}[xlabel={Epoch}, ylabel={Loss}, legend pos=north east, grid=major]
  \addplot table[x=epoch, y=train_loss, col sep=comma] {data/results.csv};
  \addplot table[x=epoch, y=val_loss, col sep=comma] {data/results.csv};
  \legend{Train, Validation}
\end{axis}
```

Always set explicit `width` and `height` to prevent overflow.

---

## Phase 5: Quality Loop (MANDATORY — iterative)

```
┌─→ 5a. Compile (2-pass XeLaTeX)
│   5b. Self-Review (structure + content + visual)
│   5c. Score (apply rubric)
│   5d. Fix all issues found
└── If score < 90 and round < 3: loop back to 5a
    If score ≥ 90 or round = 3: report to user
```

### 5a. Compilation
- 2-pass XeLaTeX
- Check: errors, overfull hbox, undefined references
- Open PDF for visual inspection

### 5b. Self-Review

*Structure:*
- [ ] Slide count matches plan (±2 tolerance)
- [ ] Logical flow: motivation → background → technique → results → summary
- [ ] No section has >4 consecutive formal slides without example or visual break
- [ ] Transition sentences between major sections

*Content density:*
- [ ] No slide has only ≤3 short bullets with no math/diagram
- [ ] Pure text-only slides ≤ 30%
- [ ] No slide exceeds upper bounds (7 bullets, 2 equations, 5 symbols, 2 boxes)

*TikZ and visuals:*
- [ ] No label-label or label-curve overlaps
- [ ] No content overflowing slide boundary
- [ ] No content overflowing inside colored boxes (visually verify every box)
- [ ] TikZ diagram fits within remaining slide space
- [ ] All marked points lie on their curves (computed via `\pgfmathsetmacro`)
- [ ] Dashed reference lines terminate at the curve
- [ ] Tables fit within slide width
- [ ] All standalone tables are centered (`\begin{center}...\end{center}`)
- [ ] No table's `\toprule` visually merges with the title bar (add `\vspace{4pt}` or text before first table)
- [ ] Font sizes in TikZ ≥ `\footnotesize`

*Notation:*
- [ ] Same symbol used consistently
- [ ] Every symbol defined before use

### 5c. Quality Score

Start at 100, deduct per issue (see main AGENTS.md Section 3).

### 5d. Fix

Fix all critical and major issues. Re-compile. Max 3 rounds.

### Post-Creation Checklist (final gate)

```
[ ] Compiles without errors
[ ] No overfull hbox > 10pt
[ ] All citations resolve
[ ] Score ≥ 90
[ ] Every definition has motivation + worked example
[ ] Max 2 colored boxes per slide
[ ] No sparse slides
[ ] TikZ diagrams visually verified — no overlaps, no overflow, all marked points on curves
[ ] Tables fit within slide boundaries, are centered, and separated from title bar
[ ] No content overflow inside colored boxes (visual PDF check)
[ ] References slide present (second-to-last)
```
