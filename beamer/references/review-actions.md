# Review Actions — Detailed Rules

This document contains the full rules for `review`, `audit`, `pedagogy`, `excellence`, and `validate` actions.

---

## `review [file]` or `proofread [file]`

Read-only report, no file edits.

### 5 Check Categories

| Category | What to check |
|----------|---------------|
| Grammar | Subject-verb, articles, prepositions, tense consistency |
| Typos | Misspellings, search-replace artifacts, duplicated words, **unreplaced placeholders** (`[name]`, `[TODO]`, `[XXX]`, template remnants) |
| Overflow | Long equations without `\resizebox`, too many items per slide |
| Consistency | Citation format (`\citet`/`\citep`), notation, terminology, box usage, **denominator consistency** across slides |
| Academic quality | Informal abbreviations, missing words, claims without citations, **ambiguous abbreviations** (same abbreviation for different terms) |

### Report Format

```
### Issue N: [Brief description]
- **Location:** [slide title or line number]
- **Current:** "[exact text]"
- **Proposed:** "[fix]"
- **Category / Severity:** [Category] / [High|Medium|Low]
```

---

## `audit [file]`

Visual layout audit. Read-only report.

### Check Dimensions

- **Overflow:** Content exceeding boundaries, wide tables/equations
- **Font consistency:** Inline size overrides, inconsistent sizes
- **Box fatigue:** 2+ boxes per slide, wrong box types
- **Spacing:** `\vspace` overuse, structural issues
- **Layout:** Missing transitions, missing framing sentences

### Spacing-First Fix Principle (priority order)

1. Reduce vertical spacing (structural changes)
2. Consolidate lists
3. Move displayed equations inline
4. Reduce image/table size with `\resizebox`
5. **Last resort:** `\footnotesize` (never `\tiny`)

---

## `pedagogy [file]`

Holistic pedagogical review. Read-only report.

### 13 Patterns to Validate

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

### Deck-Level Checks

- Narrative arc
- Pacing (max 3-4 theory slides before example)
- Visual rhythm (section dividers every 5-8 slides)
- Notation consistency
- Student prerequisite assumptions

---

## `excellence [file]`

Comprehensive multi-dimensional review.

### Parallel Dispatch

Launch these as concurrent review tasks:

1. **Visual audit** — "Read [file]. Check every slide for: overflow, font consistency, box fatigue (2+ boxes), spacing issues, missing transitions. Report per slide with severity. Follow spacing-first fix principle."
2. **Pedagogical review** — "Read [file]. Validate 13 pedagogical patterns and deck-level checks (narrative arc, pacing, visual rhythm, notation consistency). Report pattern-by-pattern with status."
3. **Proofreading** — "Read [file]. Check grammar, typos, citation consistency, notation consistency, academic quality. Report per issue with location and fix."
4. **TikZ review** (if file contains `\begin{tikzpicture}`) — "Read [file]. For every TikZ diagram: check label overlaps, geometric accuracy, visual semantics, spacing. Report per issue with exact coordinates and fixes."
5. **Domain review** (optional) — "Read [file]. Verify substantive correctness: assumptions, derivations, citation fidelity."

### Combined Report Format

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

### Quality Score Rubric

| Score | Critical | Medium | Meaning |
|-------|----------|--------|---------|
| Excellent | 0-2 | 0-5 | Ready to present |
| Good | 3-5 | 6-15 | Minor refinements |
| Needs Work | 6-10 | 16-30 | Significant revision |
| Poor | 11+ | 31+ | Major restructuring |

---

## `validate [file] [duration]`

Automated quantitative validation. Checks measurable properties without reading content.

### Checks Performed

1. **Slide count vs. duration** (if duration provided):
   ```bash
   pdfinfo FILE.pdf | grep "Pages:"
   ```
   Compare against timing allocation table. Flag if outside recommended range.

2. **Aspect ratio**:
   ```bash
   pdfinfo FILE.pdf | grep "Page size:"
   ```
   Expected: 364.19 x 272.65 pts (16:9 at 10pt) or similar 16:9 ratio.

3. **File size**: > 50 MB = warning, > 100 MB = critical.

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

### Report Format

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

---

## `visual-check [file]`

PDF-based visual verification. Converts compiled PDF to images, then reviews each slide.

### Workflow

1. **Compile** (if not already compiled):
   ```bash
   xelatex -interaction=nonstopmode FILE.tex
   ```

2. **Convert PDF to images** using PyMuPDF:
   ```python
   import fitz
   doc = fitz.open('FILE.pdf')
   zoom = 200 / 72  # 200 DPI
   matrix = fitz.Matrix(zoom, zoom)
   for i in range(len(doc)):
       page = doc.load_page(i)
       pixmap = page.get_pixmap(matrix=matrix)
       pixmap.save(f'/tmp/slide-{i+1:03d}.jpg', output='jpeg')
   doc.close()
   ```
   **Fallback** (if PyMuPDF unavailable): Read the PDF directly page by page.

3. **Per-slide inspection** checklist:
   - [ ] No text overflow at any edge
   - [ ] No content overflowing inside colored boxes
   - [ ] All text legible
   - [ ] Tables and equations fit within slide width
   - [ ] TikZ labels not overlapping
   - [ ] Consistent font sizes across similar slide types
   - [ ] Adequate contrast between text and background
   - [ ] No visual clutter

4. **Report** per issue:
   ```
   ### Slide N: [slide title]
   - **Issue:** [description]
   - **Severity:** Critical / Major / Minor
   - **Fix:** [specific recommendation]
   ```

---

## `extract-figures [pdf] [pages]`

Extract figures from paper PDFs for slide inclusion.

### Workflow

1. **Identify target figures** — locate figures in the paper. Ask user which to extract if ambiguous.

2. **Extract images** from specified pages (using PDF tools).

3. **Save to project** — decode and write each image to `figures/` directory. Naming: `fig-<descriptive-label>.png`.

4. **Generate LaTeX snippet** — ready-to-paste code for full-width, side-by-side, and subfigure layouts.

5. **Cropping guidance** — use `trim` and `clip` options to remove margins:
   ```latex
   \includegraphics[width=0.85\textwidth, trim=LEFT BOTTOM RIGHT TOP, clip]{figures/fig-LABEL.png}
   ```

### Rules

- **Always attribute** — include `{\small Source: ...}` below every extracted figure (unless user's own paper).
- **Prefer vector** — for complex multi-figure pages, consider redrawing in TikZ.
- **Resolution check** — if width < 800px, warn about pixelation.
- **Don't extract tables** — always recreate tables in LaTeX using `booktabs`.
- **Preamble dependency** — ensure `\usepackage{graphicx}` is present.
