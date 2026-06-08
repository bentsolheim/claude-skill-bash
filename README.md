# bentsolheim/public-skills

[![skills.sh](https://skills.sh/b/bentsolheim/public-skills)](https://skills.sh/bentsolheim/public-skills)

A collection of Claude Code skills authored by [@bentsolheim](https://github.com/bentsolheim) that are general enough to share publicly.

## Install

```bash
# All skills in this repo:
npx skills add bentsolheim/public-skills

# A specific skill:
npx skills add bentsolheim/public-skills -s bash
npx skills add bentsolheim/public-skills -s quarto-doc-setup
npx skills add bentsolheim/public-skills -s from-pdf-skill-builder
```

Install globally with `-g` to make a skill available to every Claude Code session, or run inside a project for a project-local install. See the [skills.sh CLI docs](https://skills.sh) for more.

## Skills

| Skill | Use when |
|---|---|
| [`bash`](skills/bash) | Writing or reviewing bash scripts and shell automation. Enforces the main-function pattern, usage docs, argument parsing, dependency checks, and error handling. |
| [`quarto-doc-setup`](skills/quarto-doc-setup) | Setting up a Quarto documentation project that renders Markdown to PDF via LaTeX templates (assessment reports, technical manuals, corporate reports, minimal-clean). |
| [`from-pdf-skill-builder`](skills/from-pdf-skill-builder) | Converting one or more PDFs (manuals, regulations, certificates) into a high-quality Claude Code skill using vision-based page-by-page extraction. |

## Versioning

Per the skills.sh ecosystem convention, this repo does **not** use git tags. Per-skill versions live in each `SKILL.md` frontmatter as `metadata.version`. Bump there when shipping a meaningful change.

## Layout

```
skills/
  <kebab-case-name>/
    SKILL.md          # required; frontmatter with name, description, metadata.version
    README.md         # optional; user-facing notes per skill
    CLAUDE.md         # optional; contributor notes per skill
    scripts/          # optional
    templates/        # optional
    examples/         # optional
```

## Repo history

This repository was renamed from `bentsolheim/claude-skill-bash` in 2026. The old URL still resolves via GitHub redirects. See [`skills/bash/README.md`](skills/bash/README.md) for the bash-skill-specific note about the 2.0.0 restructure.

## License

See individual skill folders for licensing notes (none specified inherits MIT by convention for personal use).
