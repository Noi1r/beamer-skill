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

### Highlights

- **Quality scoring system** — automated rubric (start at 100, deduct per issue) with Excellent/Good/Needs Work/Poor thresholds
- **No overlays** — no `\pause`, `\onslide`, `\only`. Uses multiple slides and color emphasis instead
- **Content density guards** — upper bounds (7 bullets, 2 equations, 5 symbols per slide) and lower bounds (every slide earns its place)
- **Box overflow detection** — Beamer suppresses overflow warnings inside blocks; the skill catches them via visual audit
- **Motivation before formalism** — every concept starts with "Why?" before "What?"
- **TikZ precision** — mathematical accuracy enforced via `\pgfmathsetmacro` (no hardcoded approximations)
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

Install [pdf-mcp](https://github.com/michaelneale/pdf-mcp) so Claude Code can read large PDFs (papers, references) directly:

```bash
pip install pdf-mcp
claude mcp add pdf-mcp --scope user pdf-mcp
```

This enables the `create` action to read and analyze full research papers before generating slides.

## Installation

### Option A: Install via `claude mcp add-skill` (Recommended)

```bash
claude skill add --url https://github.com/Noi1r/beamer-skill
```

### Option B: Manual install

Clone the repo and copy the skill directory into your Claude Code skills folder:

```bash
git clone https://github.com/Noi1r/beamer-skill.git
mkdir -p ~/.claude/skills
cp -r beamer-skill/beamer ~/.claude/skills/
```

## Usage

Once installed, the skill is triggered automatically when you mention beamer, slides, lecture, tikz, or related keywords in Claude Code.

**Create a lecture from a paper:**
```
Help me create a beamer presentation based on this paper: /path/to/paper.pdf
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
\author{Presenter: Lizheng Wang}
\institute{Shanghai Jiao Tong University}
```

To change this, either:
- Tell Claude Code your name/affiliation when creating slides, or
- Edit `beamer/SKILL.md` Section 0 (Reference Preamble) to set your own defaults

### Custom Templates

If you have a custom beamer preamble, header file, or theme, simply provide it. The skill will use yours instead of the built-in default.

## File Structure

```
beamer-skill/
├── beamer/
│   └── SKILL.md      # The skill definition
├── README.md
└── LICENSE
```

## License

MIT
