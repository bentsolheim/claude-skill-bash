---
name: quarto-doc-setup
description: Set up professional Quarto documentation projects with PDF output via LaTeX templates. Use this skill when the user wants to create a Quarto project, set up a documentation project with PDF output, configure LaTeX templates for Quarto, render markdown to professional PDF, or asks about Quarto document setup. Triggers on mentions of "quarto", "pdf template", "latex template for documents", "documentation project setup", or "professional PDF from markdown".
metadata:
  version: 1.1.0
---

# Quarto Document Project Setup

This skill sets up professional documentation projects that render Markdown to PDF using Quarto and custom LaTeX templates.

## Workflow

Follow these steps in order when setting up a new Quarto document project.

### Step 1: Check Prerequisites

Run these checks and report results to the user:

```bash
# Check Quarto
quarto --version

# Check TeX Live
pdflatex --version

# Check required LaTeX packages
for pkg in libertinus.sty fontspec.sty multirow.sty mdframed.sty zref.sty needspace.sty titlesec.sty enumitem.sty; do
  kpsewhich "$pkg" > /dev/null 2>&1 && echo "OK: $pkg" || echo "MISSING: $pkg"
done

# Check JetBrains Mono font (system font, not a TeX package)
fc-list "JetBrains Mono" | head -1 || echo "MISSING: JetBrains Mono font"
```

If Quarto is missing, tell the user to install it from https://quarto.org/docs/get-started/

If the JetBrains Mono font is missing, install it:
```bash
brew install --cask font-jetbrains-mono
```

If TeX packages are missing, provide this command:
```bash
sudo tlmgr install libertinus libertinus-fonts libertinus-otf libertine fontspec multirow mdframed zref needspace titlesec enumitem
```

### Step 2: Choose a Template

Present the user with these template options using AskUserQuestion:

1. **Corporate Report** — Professional business documents with logo support, navy blue accents, clean tables. Good for: annual reports, proposals, policy documents.

2. **Technical Manual** — Optimized for technical content with styled code blocks, admonition boxes (note/warning/tip), dark teal accents. Good for: API docs, technical specs, runbooks, SOPs.

3. **Assessment Report** — Color-coded status indicators, metadata boxes, summary tables, blue accents. Good for: compliance assessments, audits, gap analyses, security reviews.

4. **Minimal Clean** — No title page, black/gray only, wide margins, elegant typography. Good for: memos, short reports, internal documents, drafts.

### Step 3: Gather Document Metadata

Ask the user for:
- **Title** (required)
- **Subtitle** (optional)
- **Author** (optional)
- **Date** (optional, e.g., "March 2026")
- **Organization** (optional)
- **Document status** (e.g., "Draft", "Final", "Under Review")
- **Language** (optional, e.g., "norsk" for Norwegian — adds babel support)
- **Logo file path** (optional, for corporate-report template)
- **Version** (optional, for technical-manual template)

### Step 4: Create Project Files

The templates are located in this skill's `templates/` directory. Resolve the skill path by finding where this SKILL.md file lives (check `~/.claude/skills/skill-quarto-doc-setup/` for the symlink, or the original repo path). Copy the selected template and create the QMD file.

**4a. Copy the LaTeX template** to the user's project directory:
```bash
cp ~/.claude/skills/skill-quarto-doc-setup/templates/<selected-template>.tex <project-dir>/<template-name>.tex
```

**4b. Create the QMD document** with this frontmatter structure:

```yaml
---
title: "<title>"
subtitle: "<subtitle>"
author: "<author>"
date: "<date>"
organization: "<organization>"
document_status: "<status>"
short_title: "<abbreviated title for headers>"
lang: "<language>"  # optional, e.g., "norsk" — omit for English
titlepage: true
toc: true
format:
  pdf:
    template: <template-name>.tex
    number-sections: true
    keep-tex: false
---

# First Section

Your content here.
```

For the **minimal-clean** template, omit `titlepage: true` (it uses an inline title instead).

For the **technical-manual** template, add `version: "<version>"` to show a version number on the title page.

**4c. Add to .gitignore** (if a git repo):
```
# Generated PDF output
*.pdf
```

### Step 5: Render and Verify

```bash
quarto render "<document-name>.qmd" --to pdf
```

Open the generated PDF and confirm it renders correctly. If there are LaTeX errors, check for missing packages first.

## Template Details

### Shared Features (All Templates)

- KOMA-Script `scrartcl` document class (A4 paper, 11pt)
- Libertinus serif + JetBrains Mono monospace fonts (via fontspec/LuaLaTeX)
- 2.5cm margins on all sides (3cm for minimal-clean)
- Zero paragraph indent with 8pt paragraph spacing
- Styled hyperlinks matching accent color with PDF metadata
- Full Pandoc/Quarto compatibility (`\tightlist`, `Shaded`, `Highlighting`, `\pandocbounded`, syntax highlighting tokens)
- `$for(include-before)$` / `$for(include-after)$` content injection
- Section numbering responds to `number-sections` frontmatter
- Table of contents with configurable title and depth
- Professional `booktabs`-style tables with left-aligned longtables
- Customizable headers/footers with document status
- Styled inline code with background highlight
- Optional language support via babel
- Optional line spacing control

### Frontmatter Variables

All templates support these QMD YAML variables:

| Variable | Used In | Description |
|----------|---------|-------------|
| `title` | All | Document title |
| `subtitle` | All except minimal | Subtitle below title |
| `author` | All | Author name |
| `date` | All | Date shown on title page |
| `organization` | All except minimal | Organization name on title page |
| `document_status` | All | Shown in footer (e.g., "Draft") |
| `short_title` | All | Abbreviated title for page headers |
| `lang` | All | Language for babel (e.g., "norsk", "german") |
| `logo` | corporate-report | Path to logo image for title page |
| `titlepage` | All except minimal | Set `true` to generate a separate title page |
| `toc` | All | Set `true` for table of contents |
| `toc-title` | All | Custom table of contents heading |
| `toc-depth` | All | TOC depth (default: 3) |
| `linestretch` | All | Line spacing multiplier (e.g., 1.5) |
| `version` | technical-manual | Version number on title page |
| `framework` | assessment-report | Assessment framework name |
| `total_requirements` | assessment-report | Number of requirements assessed |
| `assessment_date` | assessment-report | Date of assessment |

### Template-Specific Commands

**Assessment Report** provides these LaTeX commands for use in QMD with raw LaTeX blocks:

- `\statusfully` — Green "Fully met" badge
- `\statuspartially` — Orange "Partially met" badge
- `\statusnotmet` — Red "Not met" badge
- `\assessmentsummary{fully}{partial}{not}` — Summary count table
- `\requirementmeta{title}{asset}{function}{description}` — Requirement metadata box

**Technical Manual** provides:

- `\notebox{text}` — Blue info box
- `\warningbox{text}` — Orange warning box
- `\tipbox{text}` — Green tip box

## Troubleshooting

Common issues and fixes:

1. **"Undefined control sequence" for `\pandocbounded`** — You are using an older version of the templates. Re-copy the template from this skill's directory.

2. **Missing LaTeX packages** — Run `kpsewhich <package>.sty` to check. Install with `sudo tlmgr install <package>`.

3. **Section numbers not toggling** — Ensure `number-sections: true/false` is under `format: pdf:`, not at the top level.

4. **Non-English characters broken** — Add `lang: "<language>"` to frontmatter (e.g., `lang: "norsk"` for Norwegian). The templates load babel conditionally.

5. **Overfull hbox warnings** — The templates set `\emergencystretch{3em}` to handle most cases. For persistent issues, try rewording long unbreakable strings.

6. **Images too large** — The `\pandocbounded` macro auto-scales images to fit. Use Quarto's `width` attribute for manual control: `![Caption](image.png){width=80%}`.

## Customization

To add custom LaTeX to the preamble without editing the template, use `header-includes` in frontmatter:

```yaml
format:
  pdf:
    template: corporate-report.tex
    include-in-header:
      text: |
        \usepackage{soul}
        \definecolor{mycolor}{RGB}{100, 150, 200}
```

## Example QMD

An example document is available at `examples/example-document.qmd` in this skill's directory. Use it as a reference for frontmatter and content structure.
