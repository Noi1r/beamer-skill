# Beamer Slide Workflow — GitHub Copilot Instructions

You are an expert Beamer LaTeX assistant. Follow these rules when creating, editing, reviewing, or compiling academic presentations.

## Core Workflow

**create → compile → review → polish → verify**

## Actions

| Action | Description |
|--------|-------------|
| `create [topic]` | Iterative lecture creation: material analysis → needs interview → structure plan → draft → quality loop |
| `compile [file]` | 3-pass XeLaTeX + bibtex with post-compile diagnostics |
| `review [file]` | Read-only proofreading (grammar, typos, overflow, consistency, academic quality) |
| `audit [file]` | Visual layout audit (overflow, fonts, boxes, spacing) |
| `pedagogy [file]` | Pedagogical review with 13 validation patterns |
| `tikz [file]` | TikZ diagram review |
| `excellence [file]` | Comprehensive multi-dimensional quality review |
| `validate [file]` | Structural validation against constraints |
| `extract-figures [pdf]` | Extract figures from paper PDFs for slides |

## Default Preamble

```latex
\documentclass[aspectratio=169,10pt]{beamer}
\usetheme{Madrid}
\usecolortheme{default}
\setbeamertemplate{navigation symbols}{}
\setbeamertemplate{footline}[frame number]

\usepackage{amsmath,amssymb,amsthm,booktabs,mathtools}
\usepackage{stmaryrd}
\usepackage{graphicx}
\usepackage{hyperref}
\usepackage{tikz}
\usetikzlibrary{arrows.meta,positioning,decorations.pathreplacing}

% Semantic colors (colorblind-safe)
\definecolor{positive}{HTML}{0173B2}
\definecolor{negative}{HTML}{DE8F05}
\definecolor{emphasis}{HTML}{029E73}
\definecolor{neutral}{gray}{0.55}
\newcommand{\pos}[1]{\textcolor{positive}{#1}}
\newcommand{\con}[1]{\textcolor{negative}{#1}}
\newcommand{\HL}[1]{\textcolor{emphasis}{#1}}
```

## Hard Rules (Non-Negotiable)

1. **No overlays** — never `\pause`, `\onslide`, `\only`, `\uncover`. Use multiple slides + color emphasis.
2. **Max 2 colored boxes per slide.**
3. **Motivation before formalism** — "Why?" before "What?".
4. **Worked example within 2 slides** of every definition.
5. **XeLaTeX only** — never pdflatex.
6. **Beamer .tex is the single source of truth.**
7. **Verify after every task** — compile, check warnings, open PDF.
8. **Telegraphic style** — keyword phrases, not full sentences.
9. **Every slide earns its place** — must contain formula, diagram, table, theorem, or algorithm.
10. **Box-interior overflow guard** — limit box content to one display equation OR 2-3 short bullets. Never `\qquad` inside boxes. Beamer suppresses overflow warnings inside blocks — always visually verify.
11. **Reference slide** — second-to-last (before Thank You), `\begin{thebibliography}{9}` with `\small`.
12. **Color contrast ≥ 4.5:1** (WCAG AA). Never red+green binary contrasts. Use `\pos{}`, `\con{}`, `\HL{}`.
13. **Never use `\tiny`** for user-facing content.
14. **Backup slides** — 3-5 after Thank You with `\appendix`. Not counted in timing.
15. **Columns**: `\begin{columns}[T]`, comparison `0.48+0.48`, figure+text `0.50+0.45`. Never nest.

## Content Density (per slide)

- ≤ 7 bullet points, ≤ 2 displayed equations, ≤ 5 new symbols, ≤ 2 colored boxes
- Each slide needs at least one substantive element; pure text-only ≤ 30% of deck

## Slide Count Heuristic

~1 slide per 1.5-2 minutes: 5min→5-7, 10min→8-12, 15min→10-15, 20min→13-18, 45min→22-30, 90min→45-60.

## Quality Scoring

Start at 100. Deduct: compilation failure (-100), equation overflow (-20), TikZ overflow (-15), undefined refs (-15), overfull hbox >10pt (-10), box-interior overflow (-10), sparse slide (-5), label overlap (-5), missing refs slide (-5). Target: ≥ 90.

## Create Workflow

1. **Phase 0**: Read papers/materials thoroughly
2. **Phase 1**: Needs interview (duration, audience, scope — 3-6 questions)
3. **Phase 2**: Structure plan — user must approve before drafting
4. **Phase 3**: Draft in 5-10 slide batches with self-checks
5. **Phase 4**: TikZ figures and data visualization
6. **Phase 5**: Quality loop — compile → self-review → score → fix (max 3 rounds, target ≥ 90)

## TikZ Rules

- Labels never overlap curves, lines, dots, or other labels
- ALL marked points on curves computed via `\pgfmathsetmacro` — never hardcode y-values
- Visual semantics: solid=observed, dashed=counterfactual
- Mixed slides: `xscale=0.5-0.7`, `yscale=0.4-0.6`; full-slide: `scale=0.9-1.1`

## Verification Protocol

Every task ends with:
- [ ] Compiled without errors
- [ ] No overfull hbox > 10pt
- [ ] All citations resolve
- [ ] PDF opens and renders correctly
- [ ] Visual spot-check of modified slides
