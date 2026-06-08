# Vision-extraction subagent prompt template

Copy the template below and fill in the `{{PLACEHOLDERS}}` for the specific chapter, then pass as the prompt to a `general-purpose` subagent.

The template is structured in seven blocks: context, inputs, output, process, markdown structure, faithfulness rules, final report. **Do not remove any block** — the subagent has no memory of the parent conversation and depends on every instruction in this prompt.

---

## Placeholders

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `{{DOCUMENT_TYPE}}` | One-line description of what the PDF is | `Yamaha F80B/F80C/F100D outboard motor owner's manual` |
| `{{LANGUAGE}}` | Source document language | `English` |
| `{{ACCURACY_STAKES}}` | Why accuracy matters for this content | `Wrong torques or capacities could cause real harm` |
| `{{PDF_PATH}}` | Absolute path to bundled PDF | `/path/to/skill/manual.pdf` |
| `{{CHAPTER_NUMBER}}` | Chapter number, or omit for unnumbered | `5` |
| `{{CHAPTER_TITLE}}` | Original chapter title verbatim | `Characteristics and use` |
| `{{PAGE_COUNT}}` | Number of pages | `16` |
| `{{PHYSICAL_PAGE_RANGE}}` | Physical PDF pages, e.g. `15-20` | `12-27` |
| `{{LOGICAL_PAGE_INFO}}` | Logical page mapping if different, else omit | ` (logical page 9-14)` |
| `{{OUTPUT_PATH}}` | Absolute path where the markdown should be written | `/path/to/skill/chapters/05-characteristics-and-use.md` |
| `{{DIAGRAMS_FOLDER}}` | Absolute path to existing diagrams folder | `/path/to/skill/diagrams/` |
| `{{EXISTING_DIAGRAMS_LIST}}` | Bulleted list of files in diagrams folder with short descriptions | (see below) |
| `{{TMP_PREFIX}}` | Unique `/tmp` prefix for this subagent's PNGs | `y56-c05-p` |
| `{{SOURCE_LINE_FORMAT}}` | How to format the source citation | `page 12-27` or `physical page 12-27 (logical page 9-14)` |
| `{{SPECIAL_NOTES}}` | Optional chapter-specific guidance | "This chapter has the spec tables — be especially careful with numbers" |

### Example `{{EXISTING_DIAGRAMS_LIST}}` format

```
  - `components.png` — full motor diagram with numbered legend (page 21)
  - `greasing-points.png` — 6 lubrication points (page 80)
  - `fuse-box.png` — fuse positions 1-7 in electrical cover (page 95)
```

---

## Template

```
Convert ONE chapter of a {{LANGUAGE}} {{DOCUMENT_TYPE}} from PDF to structured markdown for a Claude Code skill. **Accuracy is critical** — {{ACCURACY_STAKES}}.

**Inputs:**
- PDF: `{{PDF_PATH}}`
- Chapter: **{{CHAPTER_NUMBER}} "{{CHAPTER_TITLE}}"** ({{PAGE_COUNT}} pages: physical pages {{PHYSICAL_PAGE_RANGE}}{{LOGICAL_PAGE_INFO}})
- Existing diagrams folder: `{{DIAGRAMS_FOLDER}}` containing:
{{EXISTING_DIAGRAMS_LIST}}

**Output:** Write the final structured markdown to:
`{{OUTPUT_PATH}}`

## Process

For each page in physical range {{PHYSICAL_PAGE_RANGE}}:

1. Render page as a high-resolution PNG to /tmp:
   ```
   pdftoppm -png -r 200 -f N -l N "{{PDF_PATH}}" /tmp/{{TMP_PREFIX}}
   ```
   (Creates `/tmp/{{TMP_PREFIX}}-NN.png` with zero-padded number)
2. **View the PNG using the Read tool** — you are multimodal and can see images directly.
3. Transcribe the page content into structured markdown, appending to your work-in-progress draft.
4. When all pages are processed, write the complete markdown to the output path.
5. Delete the temp PNGs: `rm /tmp/{{TMP_PREFIX}}-*.png`

## Markdown structure

The first lines of the file must be exactly:
```
# {{CHAPTER_TITLE}}

**Source:** `<source-filename>` {{SOURCE_LINE_FORMAT}}

---

```

For the body, mirror the document's own hierarchy:

- The document uses section numbers like "5.1", "5.2" — convert each to a `## {{CHAPTER_NUMBER}}.N Title` heading.
- Sub-sections become `### `.
- Bullets in the PDF → markdown `-` lists.
- Numbered steps → markdown `1.`, `2.`, ... lists.
- Tables (specifications, schedules, comparisons) → real markdown tables with proper alignment. Be careful with multi-row entries where a label spans rows — keep them logically grouped.
- **WARNING!**, **CAUTION!**, **DANGER!**, **NOTE!**, **NOTICE**, **TIP**, **IMPORTANT**, **PRECAUTION!** boxes → blockquote with bold prefix:
  ```
  > **WARNING!** Do not exceed the maximum recommended load.
  ```
- Single-line emphasis inline (e.g. "NOTE!" mid-paragraph) → keep as `**NOTE!**` in flowing text.

## Two-column handling

Several pages may have two-column text layouts. **Read each column top-to-bottom completely before moving to the next column.** Never interleave lines from left and right columns. If the section continues on the next page or the next column, treat it as one continuous section in the markdown — no mid-section page break markers.

## Diagram referencing

When a page contains a figure/diagram:

1. **If the figure matches one of the existing PNG files listed above** (judge by what you see in the page vs. the file's description, not by trying to match names), embed it inline at the right point in the text:
   ```
   ![<short description>](../diagrams/<filename>.png)
   ```
   Use a short, descriptive alt text.

2. **If the page has a useful figure NOT in the existing folder** (numbered legend diagrams, layout diagrams, procedure illustrations, charts), do NOT try to extract it yourself. Instead, insert a placeholder line:
   ```
   *[Figure: <description of what it shows>, see source.pdf page <N>]*
   ```
   AND add it to your "Suggested new diagrams" report at the end with: `page N | suggested-slug.png | description`.

3. **For tiny inline figures (a single icon, a single arrow indicator, decorative bullet glyph)**, just describe in prose: "Figure shows the fuel filler cap on the starboard side, marked 'FUEL'."

## Faithfulness rules

- **Do not translate.** The document is {{LANGUAGE}}; the markdown stays {{LANGUAGE}}.
- **Do not paraphrase or summarize.** Transcribe the document's wording as closely as possible.
- **Numbers, dates, currencies, units must be exact.** Re-read each one. Examples that demand re-read: torques ("25.0 Nm" not "2.5 Nm"), capacities ("0.670 L" not "0.067 L"), policy/order numbers, dates ("09.10.2025" stays in that format), units (mm vs in, L vs US qt, Nm vs ft-lb).
- **Preserve original typos verbatim.** Do not auto-correct the document's author.
- **If unreadable**, write `[unreadable: see source.pdf page N]` rather than guess. Do not fabricate values.
- **Do not insert page break markers** in the middle of a flowing section.
- **Skip page chrome:** logos, page numbers like "12 (36)", repeating headers/footers, book identifiers like `U6D777E0.book Page 39 Wednesday, April 7, 2010 8:33 AM`.
{{SPECIAL_NOTES}}

## Final report (under 200 words, or 300 for chapters over 15 pages)

1. **Pages processed** (should be {{PAGE_COUNT}}).
2. **Suggested new diagrams** as: `page N | suggested-slug.png | description of what it shows`. List each figure you flagged with a placeholder, plus any useful diagram you saw but didn't placeholder.
3. **Uncertain transcriptions** — be explicit about any value or word you weren't 100% sure on. Note the page and what looked ambiguous.
4. **Quality verdict** — does the resulting markdown read as a clean, structured single-column version of the chapter, with all data exact?

Do not commit, push, or modify other files. Do not delete the input PDF or existing diagrams.
```

---

## Notes on customizing the template

- **For documents with no chapter numbers** (e.g. magazines, certificates), drop the chapter-number heading convention and let the subagent use the document's own section structure (`## Section Title`).
- **For data-dense chapters** (specifications, pricing tables), add a `{{SPECIAL_NOTES}}` paragraph emphasizing tables and unit precision.
- **For procedure-heavy chapters** (maintenance steps, emergency procedures), add `{{SPECIAL_NOTES}}` emphasizing numbered-list preservation and warning callouts.
- **For chapters with many figures** (component overviews, instrument panels), list all expected matches in `{{EXISTING_DIAGRAMS_LIST}}` with their page numbers so the subagent maps them correctly.
- **For chapters with little or no figures** (warranty, definitions), the diagram section of the template still applies — the subagent will just have nothing to embed or flag.
