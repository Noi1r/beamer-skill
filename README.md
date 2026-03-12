# Beamer Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for creating, compiling, reviewing, and polishing academic **Beamer LaTeX** presentations.

Full lifecycle: **create → compile → review → polish → verify.**

## Features

| Action | Description |
|--------|-------------|
| `create [topic]` | Collaborative, iterative lecture creation with phase gates (material analysis → needs interview → structure plan → draft → quality loop) |
| `compile [file]` | 3-pass XeLaTeX + bibtex with post-compile diagnostics |
| `review [file]` | Read-only proofreading report (grammar, typos, overflow, consistency, academic quality) |
| `audit [file]` | Visual layout audit (overflow, fonts, boxes, spacing) |
| `pedagogy [file]` | Holistic pedagogical review with 13 validation patterns |
| `tikz [file]` | TikZ diagram review and SVG extraction |
| `excellence [file]` | Comprehensive multi-dimensional quality review |
| `devils-advocate [file]` | Challenge slide design with pedagogical questions |
| `visual-check [file]` | PDF-based visual verification of compiled slides |
| `validate [file]` | Structural validation against skill constraints |
| `extract-figures [pdf]` | Extract figures from paper PDFs for direct inclusion in slides |

### Highlights

- **Quality scoring system** — automated rubric (start at 100, deduct per issue) with Excellent/Good/Needs Work/Poor thresholds
- **No overlays** — no `\pause`, `\onslide`, `\only`. Uses multiple slides and color emphasis instead
- **Content density guards** — upper bounds (7 bullets, 2 equations, 5 symbols per slide) and lower bounds (every slide earns its place)
- **Box overflow detection** — Beamer suppresses overflow warnings inside blocks; the skill catches them via visual audit
- **Motivation before formalism** — every concept starts with "Why?" before "What?"
- **TikZ precision** — mathematical accuracy enforced via `\pgfmathsetmacro` (no hardcoded approximations)
- **Semantic color system** — colorblind-safe palette (`\pos{}`, `\con{}`, `\HL{}`) with WCAG AA contrast (≥ 4.5:1)
- **Figure extraction** — pull figures directly from paper PDFs via `pdf-mcp`, ready for `\includegraphics`
- **Timing allocation** — built-in slide-count heuristics for 5min lightning talks through 90min lectures
- **Columns & layout rules** — enforced `columns[T]` patterns with gap/width constraints
- **Backup slides** — automatic appendix section for anticipated Q&A
- **Algorithm & code support** — `algorithm2e`, `listings`, `pgfplots` integration with per-slide line limits
- **XeLaTeX only** — modern font handling, 16:9 aspect ratio, 10pt default

## Prerequisites

### TeX Distribution

A full TeX distribution with XeLaTeX is required:

```bash
# macOS
brew install --cask mactex

# Ubuntu/Debian
sudo apt install texlive-full

# Arch
sudo pacman -S texlive
```

### pdf-mcp (Recommended)

Install [pdf-mcp](https://github.com/jztan/pdf-mcp) so Claude Code can read large PDFs (papers, references) and extract figures directly:

```bash
pip install pdf-mcp
claude mcp add pdf-mcp --scope user pdf-mcp
```

If you use a SOCKS proxy, also install `socksio` (required by `httpx` for SOCKS support):

```bash
pip install socksio
```

This enables the `create` action to analyze research papers and `extract-figures` to pull figures for slide inclusion.

## Installation

Clone the repo and copy the skill directory into your Claude Code skills folder:

```bash
git clone https://github.com/Noi1r/beamer-skill.git
mkdir -p ~/.claude/skills
cp -r beamer-skill/beamer ~/.claude/skills/
```

Then restart Claude Code. The skill will be automatically detected.

## Usage

Once installed, the skill is triggered automatically when you mention beamer, slides, lecture, tikz, or related keywords in Claude Code.

**Create a lecture from a paper:**
```
Help me create a beamer presentation based on this paper: /path/to/paper.pdf
```

**Extract figures from a paper:**
```
Extract figures from /path/to/paper.pdf for my slides
```

**Compile slides:**
```
Compile my slides: /path/to/slides.tex
```

**Full quality check:**
```
Run excellence review on /path/to/slides.tex
```

**Proofread only:**
```
Proofread /path/to/slides.tex
```

## Customization

### Default Presenter

The skill defaults to:
```latex
\author{Presenter: [name]}
\institute{Shanghai Jiao Tong University}
```

To change this, either:
- Tell Claude Code your name/affiliation when creating slides, or
- Edit `beamer/SKILL.md` Section 0 (Reference Preamble) to set your own defaults

### Custom Templates

If you have a custom beamer preamble, header file, or theme, simply provide it. The skill will use yours instead of the built-in default.

## Examples

The `example/` directory contains real-world examples generated entirely by this skill:

| Example | Topic | Type |
|---------|-------|------|
| `zkagent_slides` | zkAgent — zero-knowledge proof agents | Theoretical CS / cryptography |
| `slides` | TWIST1⁺ FAP⁺ fibroblasts in Crohn's disease | Basic research (with extracted figures) |
| `slides_EP` | Endoscopic papillectomy outcomes | Clinical retrospective study |

Each example includes the source paper (PDF), the generated `.tex`, and the compiled `.pdf`. The `figures/` directory contains images extracted from the source papers via `extract-figures`.

## Benchmark: With Skill vs. Without Skill

Automated evaluation comparing Claude Code **with** the beamer skill against a **baseline** (no skill). Each test was run by an independent subagent; assertions check objective, verifiable properties of the output.

### Test 1 — `create`: 10-min talk from a cryptography paper

| Assertion | With Skill | Baseline |
|-----------|:----------:|:--------:|
| .tex file generated | Pass | Pass |
| `aspectratio=169`, `10pt` | Pass | Pass |
| No overlay commands | Pass | Pass |
| Slide count in 8–12 range | Pass (11) | Pass (12) |
| References slide (`thebibliography`) | Pass | **Fail** |
| Backup slides after `\appendix` | Pass | **Fail** |
| Semantic colors (`\pos`, `\con`, `\HL`) | Pass | **Fail** |
| **Pass rate** | **7/7 (100%)** | **4/7 (57%)** |

### Test 2 — `review` + `validate`: proofread an existing slide deck

| Assertion | With Skill | Baseline |
|-----------|:----------:|:--------:|
| Review report generated | Pass | Pass |
| 5-category classification | Pass | **Fail** |
| Standard issue format (Location / Current / Proposed / Severity) | Pass | **Fail** |
| Separate validation report | Pass | **Fail** |
| Structured validation table (slide count, aspect ratio, hard rules) | Pass | **Fail** |
| Original `.tex` unmodified (read-only) | Pass | Pass |
| **Pass rate** | **6/6 (100%)** | **2/6 (33%)** |

### Summary

| Metric | With Skill | Baseline | Delta |
|--------|:----------:|:--------:|:-----:|
| Overall pass rate | 13/13 (100%) | 6/13 (46%) | **+54 pp** |
| Mean tokens | ~65k | ~50k | +30% |
| Mean wall time | ~230s | ~130s | +73% |

> The skill guarantees structural compliance (references, backups, semantic colors, formatted reports) that the baseline consistently misses. Token/time overhead is moderate (~30%/~73%) relative to the quality gain.

## File Structure

```
beamer-skill/
├── beamer/
│   └── SKILL.md          # The skill definition
├── example/               # Real-world examples
│   ├── 199.pdf            # Source paper (zkAgent)
│   ├── zkagent_slides.*   # Generated slides
│   ├── slides.*           # Crohn's disease fibrosis
│   ├── slides_EP.*        # Endoscopic papillectomy
│   └── figures/           # Extracted paper figures
├── README.md
└── LICENSE
```

## License

MIT
