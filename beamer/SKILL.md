---
name: beamer
description: |
  Beamer LaTeX slide workflow: create, compile, review, and polish academic presentations.
  Use when working on any Beamer .tex slide deck — creation, editing, compilation, proofreading,
  visual audit, pedagogical review, TikZ diagrams, or comprehensive quality check.
  Trigger words: beamer, slides, lecture, tikz, compile latex, proofread slides, slide review.
argument-hint: "[action] [file] — actions: create, compile, review, audit, proofread, tikz, excellence"
allowed-tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "Agent", "Task"]
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
\usepackage{hyperref}
\usepackage{tikz}
\usetikzlibrary{arrows.meta,positioning,decorations.pathreplacing}
```

**Rules:**
- **Always `10pt`** — `11pt` or `12pt` produces oversized, sparse slides.
- **Always `aspectratio=169`** — modern projectors are 16:9.
- **Default presenter**: `\author{Presenter: Lizheng Wang}` and `\institute{Shanghai Jiao Tong University}`. Override only if user specifies otherwise.
- If user provides a custom preamble, header file, or theme: use theirs.
- Add domain-specific macros (e.g. `\newcommand{\F}{\mathbb{F}}`) as needed.

---

## 1. HARD RULES (Non-Negotiable)

1. **No overlays** — never use `\pause`, `\onslide`, `\only`, `\uncover`. Use multiple slides for progressive builds, color emphasis for attention.
2. **Max 2 colored boxes per slide** — more dilutes emphasis. Demote transitional remarks to plain italic.
3. **Motivation before formalism** — every concept starts with "Why?" before "What?".
4. **Worked example within 2 slides** of every definition.
5. **XeLaTeX only** — never pdflatex.
6. **Beamer .tex is the single source of truth** — TikZ diagrams, content, notation all originate here.
7. **Verify after every task** — compile, check warnings, open PDF.
8. **Telegraphic style** — keyword phrases, not full sentences. Slides are speaker prompts, not manuscripts. Exception: framing sentences that set up a definition or transition.
9. **Every slide earns its place** — each slide must contain at least one substantive element (formula, diagram, table, theorem, or algorithm). A slide with only 3 short bullets and nothing else must be merged or enriched.
10. **Box-interior overflow guard** — `alertblock`, `exampleblock`, and `block` environments add internal padding (~15% less width, ~12-16pt extra height for title bar + vertical padding). Content that fits on a bare slide can overflow inside a box in **both directions**. Rules:
    - **Vertical overflow** (most common, hardest to detect): display math (`\[ \]`) + text below it inside a single box easily exceeds vertical capacity. Limit box content to **one display equation OR 2-3 short bullet items** — not both. Never use aggressive `\vspace{-Xpt}` inside a box; it pulls bottom content past the border.
    - **Horizontal overflow**: never use `\qquad` inside a box; use `\quad` or `,`. If a display equation is wider than ~70% of `\textwidth` on a bare slide, reformat before placing inside a box.
    - **Beamer suppresses overfull warnings inside blocks** — zero compile warnings does NOT guarantee no visual overflow. Always visually verify every box in the PDF.
11. **Reference slide** — the second-to-last slide (before Thank You) must be a **References** slide listing key cited works. Use `\begin{thebibliography}{9}` with `\small`. Include the primary paper and 3-5 most relevant references.

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

**Content-driven questions** (derive from Phase 0, ask as many as needed):
- **Prerequisite knowledge**: List concrete technical dependencies identified in Phase 0. Ask which ones the audience knows. E.g., "The paper builds on sumcheck and polynomial commitments. Should I review these?" — not "familiar with basic algebra?".
- **Content scope**: Offer the paper's actual components as options. Ask which to emphasize, skip, or briefly mention.
- **Depth vs. breadth**: If the paper has both intuitive overview and detailed constructions, ask which the user prefers.
- **Paper-specific decisions**: E.g., if the paper compares two constructions, ask whether to present both equally or focus on one.

**Guidelines:**
- Options should come from the paper's actual content, not generic templates.
- 3-6 questions total; don't over-ask, don't under-ask.
- If something is obvious from context (e.g., user said "讨论班"), infer rather than ask.

**Slide count heuristic**: ~1 slide per 1.5-2 minutes (20min → 10-13 slides, 45min → 22-30 slides, 90min → 45-60 slides).

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
- **Use `\textbf{}` for key terms** on first introduction; use `\textcolor{green!60!black}{...}` for positive properties and `\textcolor{red}{...}` for drawbacks/limitations.

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
- After each batch, self-check: notation consistency, density constraints, motivation-before-formalism
- Continue to next batch only after current batch passes self-check

#### Phase 4: Figures

- TikZ diagrams in Beamer source (single source of truth)
- Apply TikZ quality standards (see Section 2.6)
- R/Python scripts if needed

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
- 2-pass XeLaTeX compilation
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
- [ ] Tables fit within slide width
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
| Critical | Undefined control sequence / citation | -15 |
| Critical | Overfull hbox > 10pt | -10 |
| Major | Content overflow inside colored box (vertical or horizontal, visual-only) | -10 per box |
| Major | Sparse slide (≤3 items, no math/diagram) | -5 per slide |
| Major | TikZ label overlap | -5 |
| Major | Missing references slide | -5 |
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
[ ] TikZ diagrams visually verified — no overlaps
[ ] Tables fit within slide boundaries
[ ] No content overflow inside colored boxes (visual PDF check)
[ ] References slide present (second-to-last, before Thank You)
```

### 2.3 `review [file]` or `proofread [file]`

Read-only report, no file edits.

**5 check categories:**

| Category | What to check |
|----------|---------------|
| Grammar | Subject-verb, articles, prepositions, tense consistency |
| Typos | Misspellings, search-replace artifacts, duplicated words |
| Overflow | Long equations without `\resizebox`, too many items per slide |
| Consistency | Citation format (`\citet`/`\citep`), notation, terminology, box usage |
| Academic quality | Informal abbreviations, missing words, claims without citations |

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
- **Mathematical accuracy of plotted points**: when marking a point at the intersection of two curves, or on a curve, NEVER hardcode approximate coordinates. Always compute the exact position using `\pgfmathsetmacro` (solve the equation analytically first, then encode the formula). For curve intersections, solve the system algebraically, then use the exact expression in TikZ. Example:
  ```latex
  % Intersection of y=0.5x and y=2.5/x+0.3:
  % 0.5x = 2.5/x + 0.3 => x^2 - 0.6x - 5 = 0 => x = (0.6+sqrt(20.36))/2
  \pgfmathsetmacro{\xint}{(0.6 + sqrt(20.36))/2}
  \pgfmathsetmacro{\yint}{0.5*\xint}
  \fill (\xint, \yint) circle (3pt);
  ```
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
[ ] Consistent dot style (solid=observed, hollow=counterfactual)
[ ] Consistent line style (solid=observed, dashed=counterfactual)
[ ] Arrow annotations: FROM label TO feature
[ ] Axes extend beyond all data points
[ ] Labels legible at presentation size
[ ] Minimum spacing between labels and graphical elements
[ ] Points on curves/intersections computed via \pgfmathsetmacro (no hardcoded approximations)
```

### 2.7 `excellence [file]`

Comprehensive multi-dimensional review. Runs checks in parallel:

1. **Visual audit** — overflow, fonts, boxes, spacing
2. **Pedagogical review** — 13 patterns + deck-level analysis
3. **Proofreading** — grammar, typos, consistency
4. **TikZ review** — (if file contains TikZ)
5. **Domain review** — (if domain reviewer configured)

**Quality score rubric:**

| Score | Critical issues | Medium issues | Meaning |
|-------|----------------|---------------|---------|
| Excellent | 0-2 | 0-5 | Ready to present |
| Good | 3-5 | 6-15 | Minor refinements |
| Needs Work | 6-10 | 16-30 | Significant revision |
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
