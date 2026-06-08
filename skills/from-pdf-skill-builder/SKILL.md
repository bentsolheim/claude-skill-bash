---
name: from-pdf-skill-builder
description: Build a high-quality Claude Code skill from one or more PDFs using vision-based page-by-page extraction. Use when converting reference material (technical manuals, certificates, policy documents, magazines, books, regulations) into a skill with structured markdown chapters, bundled diagrams, and a navigable SKILL.md. Triggers on requests like "convert this PDF to a skill", "make a skill from these manuals", "build a knowledge base from these documents", "vision-extract this PDF into a skill", or when the user has decided to turn PDF reference material into a Claude Code skill.
metadata:
  version: 1.0.0
---

# Build a skill from PDF(s) via vision

This skill is the playbook for converting one or more PDF reference documents into a high-quality Claude Code skill. The output is a skill folder where:

- Source PDFs are bundled (the skill is self-contained and portable)
- Each chapter/section is a structured markdown file (real headings, tables, blockquotes, inline diagram references)
- Diagrams are extracted as PNG and named by content
- `SKILL.md` has frontmatter triggers, navigation, quick-lookup tables, and "common questions → file" pointers

The defining technique is **vision extraction via a general-purpose subagent per chapter**. Reading PDF pages as images and writing structured markdown gives dramatically higher quality than `pdftotext` for any document with tables, multi-column layouts, callouts, or diagrams.

## Target skill structure

```
<skill-name>/
├── SKILL.md              # Frontmatter + navigation + quick-lookup
├── <source>.pdf          # Bundled original (or multiple)
├── kapitler/             # Use original-language word: "chapters", "kapitler", "kapitel", "luvut"...
│   ├── 01-<slug>.md
│   ├── 02-<slug>.md
│   └── ...
└── diagrammer/           # Match source-language word as for chapters
    ├── <content-name>.png
    └── ...
```

Optional subfiles for related but distinct topics that don't fit the chapter model (e.g. a verbatim certificate `forsikringsbevis.md` alongside curated `redningsselskapet.md`).

## Process

### 1. Setup

1. Choose a kebab-case skill name that names the **domain**, not the document title (e.g. `motor-yamaha-f100`, not `yamaha-f100-owners-manual-2010`).
2. Create folder structure: `SKILL.md`, `kapitler/`, `diagrammer/`.
3. Copy source PDF(s) into the skill folder. Rename to descriptive slugs (e.g. `manual.pdf`, `policy.pdf`, `loggen.pdf`) — not their original filenames.

### 2. Discover document structure

Before extracting, identify chapters/sections:

- Read the TOC page or sample a few pages with `pdftotext -f N -l N <pdf> -` to get titles and ranges.
- Note any difference between **physical** PDF pages (what `pdftoppm` uses) and **logical** pages (what the document prints). Vehicle/machinery manuals often have a 6-page offset; magazines and books usually match.
- Map each chapter to a physical page range. Save this mapping — every subagent needs its range.

### 3. Vision-extract chapters in parallel

For each chapter, spawn one `general-purpose` subagent using the template at [`templates/subagent-vision-prompt.md`](templates/subagent-vision-prompt.md). Fill in the placeholders, then invoke.

**Send all subagent invocations in a single message** so they run in parallel. Wall-clock time is bounded by the longest chapter (typically 10-15 minutes for a 20+ page chapter, 1-3 minutes for short chapters).

Token cost is roughly 40-70k per chapter; budget accordingly for large documents.

Each subagent returns a brief report:
- Pages processed
- Suggested new diagrams (with proposed slug and description)
- Uncertain transcriptions to verify
- Quality verdict

Collect the diagram suggestions across all subagents.

### 4. Extract diagrams

Render suggested diagrams (and any obvious whole-page figures you want available):

```bash
pdftoppm -png -r 200 -f N -l N source.pdf /path/to/diagrammer/<content-name>
```

- 200 DPI is the default — good balance of quality and file size.
- 300 DPI for verification of suspect text or fine detail.
- Name PNGs by **what the figure shows**, not its page number. `greasing-points-f80bet.png`, not `page-80.png`.
- `pdftoppm` zero-pads the resulting filename with 2 or 3 digits depending on the page range — handle both when renaming:

```bash
# Strip the -NN or -NNN suffix pdftoppm adds:
for f in *.png; do
  new=$(echo "$f" | sed -E 's/-[0-9]{2,3}\.png$/.png/')
  [ "$f" != "$new" ] && mv "$f" "$new"
done
```

Verify each rendered diagram by opening it with the `Read` tool — confirm it shows what the chapter file references.

### 5. Build SKILL.md

Use the skeleton at [`templates/skill-md-skeleton.md`](templates/skill-md-skeleton.md).

Key choices:

- **Description language:** write the frontmatter `description` in the **user's** likely query language, not the document's language. A Norwegian user querying about an English manual will type in Norwegian; the description must match those queries to trigger.
- **Description content:** list explicit trigger phrases — synonyms, specific identifiers (policy numbers, model codes), product names. The combined `description` + `when_to_use` is capped at 1,536 characters, so be dense.
- **Quick-lookup tables:** put the most-asked facts directly in SKILL.md (specs that change rarely, key numbers, contacts). These avoid loading a full chapter for a one-line answer.
- **Navigation table:** every chapter and diagram should be linked from SKILL.md so users can jump straight in. Include short descriptions of what each contains.
- **Common questions → file:** add a table like "Where is X? → [chapter Y]". This is a learned-question map; populate from real or anticipated queries.

Keep SKILL.md **under 500 lines** per Anthropic guidance. Move detail to chapter files or topical subfiles.

### 6. Validate

- **Spot-check 15-25 critical datapoints** from the chapter files against the PDF, focusing on safety- or finance-critical numbers (torques, capacities, deductibles, sums, fuse ratings, dates, prices).
- **Verify all markdown links resolve.** Pattern:
  ```bash
  cd <skill-folder>
  grep -hoE '\(kapitler/[^)]+\)|\(diagrammer/[^)]+\)' SKILL.md kapitler/*.md | \
    sed -E 's/^\(//; s/\)$//' | sort -u | \
    while read p; do [ -f "$p" ] && echo "OK $p" || echo "MISS $p"; done
  ```
- **Test the trigger.** Restart Claude (or open a new session) and ask 3-5 sample questions matching the description. Verify the skill activates and gives correct answers.

### 7. Correct OCR errors

When subagent reports flag uncertain transcriptions, re-render the suspect page at 300 DPI and verify with the `Read` tool. Apply targeted edits to the chapter file. See "OCR error patterns" below.

## Conventions

### Faithfulness to source

- **Do not translate.** Source-language content stays in source language. Only the SKILL.md *navigation layer* (descriptions, quick-lookup labels) uses the user's language.
- **Do not paraphrase or summarize.** Chapter files are verbatim transcriptions.
- **Preserve original typos.** Do not "correct" the source author. (This is part of validation honesty.)
- **`[unreadable: see source.pdf page N]`** rather than guess.

### Filename language

Match the source document's language:

| Source language | Chapter folder | Diagram folder |
|-----------------|----------------|----------------|
| English | `chapters/` | `diagrams/` |
| Norwegian | `kapitler/` | `diagrammer/` |
| German | `kapitel/` | `diagramme/` |
| Swedish | `kapitel/` | `diagram/` |

Within each folder, use English filenames matching the document's own chapter/diagram terminology. So an English boat manual's chapter 5 "Characteristics and use" becomes `chapters/05-characteristics-and-use.md`.

### Bundling

Always bundle the source PDF(s) into the skill folder. The skill is then portable and self-verifiable — anyone can compare extracted markdown against the source. Symlinks are brittle; copy the files.

## Quality rules

### Vision rendering

- **Default 200 DPI.** Higher DPI = sharper text but larger PNGs and more processing.
- **300 DPI for verification** of dense small text or where subagent flagged uncertainty.
- **Re-render is cheap.** When in doubt, look at the page image again.

### Two-column / multi-column layouts

This is the single biggest reason to use vision instead of `pdftotext`. The subagent prompt explicitly instructs: read each column top-to-bottom completely before moving to the next column, never interleave.

### Numbers and exact values

Safety- and finance-critical numbers must be exact. For each number transcribed, the subagent should be able to re-read it from the page image without ambiguity. The prompt includes "re-read each torque/capacity/voltage" and "preserve units exactly (mm vs in, L vs US qt, Nm vs ft-lb)".

### OCR error patterns

Vision occasionally misreads visually-similar glyphs. Watch for:

| Misreading | Actual | Context |
|------------|--------|---------|
| `benner` | `tenner` | Norwegian "ignites" |
| `From` | `Pump` | Imperative verb start of sentence |
| `runde` | `runder` | Trailing letter dropped |
| `dieksom` | `dersom` | Mid-word confusion |
| `1.0.1.1 mm` | `1.0–1.1 mm` | Hyphen vs en-dash |
| Latin-glyph confusions: `0/O`, `1/l/I`, `b/h/k`, `m/n/rn` | various | Especially in serif fonts at low DPI |

When the subagent reports an uncertain word, re-render at 300 DPI and trust the image.

### Page chrome

Strip from every chapter:
- Page numbers (e.g. "12 (36)")
- Repeating headers (chapter title bars, magazine name)
- Repeating footers (URL, organization name)
- Logos and decorative elements
- Book/file identifiers (e.g. `U6D777E0.book Page 39 Wednesday, April 7, 2010 8:33 AM`)

Form feed characters (`\f`) prefix some page headers in `pdftotext` output and break naive regex anchored with `^`. Not an issue for vision extraction directly, but if you ever fall back to text-based cleanup, run `tr -d '\f'` first.

### Diagram embedding

In each chapter file, embed an inline diagram reference at the natural reading point:

```markdown
![Short description of what the figure shows](../diagrammer/<filename>.png)
```

For diagrams that exist but the subagent's chapter doesn't naturally reference, list them in SKILL.md's diagram-overview table only.

For very small inline figures (single icons, single arrows), the subagent should describe in prose rather than insert a placeholder.

For useful figures the subagent identifies but that don't exist as PNGs yet, the subagent should insert a placeholder line:

```markdown
*[Figure: <description>, see source.pdf page <N>]*
```

…and flag it in its final report. You then extract those PNGs in step 4 and either replace the placeholders or leave them as references.

## When NOT to use vision

Use simpler tools when:

- **The document is pure prose, single-column, no figures, no tables.** Plain `pdftotext` (no flags) handles this in seconds.
- **A single-page document.** Manual transcription may be faster than spinning up a subagent.
- **The document is already structured markdown or HTML.** Convert directly without vision.

Even in these cases, the **structure** conventions (SKILL.md navigation, quick-lookup, faithfulness) still apply.

## Multi-PDF skills

When a skill covers several related PDFs:

- Bundle all PDFs in the skill folder with descriptive slugs.
- Either keep separate chapter folders per source (`chapters-manual/`, `chapters-policy/`) or merge into one `chapters/` with a source-prefixed naming scheme.
- SKILL.md navigation lists each PDF and its chapters/sections explicitly.
- Cross-reference between sources where they touch (e.g. policy mentions a procedure documented in manual).

## Templates and references

- **Vision subagent prompt template:** [`templates/subagent-vision-prompt.md`](templates/subagent-vision-prompt.md). Copy, fill in placeholders, and pass as the subagent prompt.
- **SKILL.md skeleton:** [`templates/skill-md-skeleton.md`](templates/skill-md-skeleton.md). Copy and adapt for the specific skill.

## Notes on subagent usage

- **Subagent has zero shared memory** with the parent. Every prompt is self-contained: PDF path, page range, output path, available diagrams, faithfulness rules, expected report format.
- **Use `general-purpose`** as the subagent type. It has full tool access including `Read` (which handles PNGs as images) and `Bash` (for `pdftoppm` and cleanup).
- **One subagent per chapter** — not per page, not per document. Per-chapter scope keeps each task bounded (5-50 pages) and parallelizable.
- **Send all subagent invocations in one message** to run them in parallel. Sequential is much slower.
- **Each subagent uses a unique `/tmp/<prefix>-` for its rendered PNGs** to avoid collisions when running in parallel.
- **Subagents clean up their own temp files** as the last step.

## Iteration

Skills mature through revision:

- **First pass** often surfaces things you didn't know about the document.
- **Keep a backup** of the first extraction during conversion (e.g. `chapters-raw/`) until you've validated quality, then delete.
- **Re-run individual chapter subagents** when you find quality issues — much cheaper than re-running everything.
- **SKILL.md grows** as you encounter more "common questions" that should be in the quick-lookup.
