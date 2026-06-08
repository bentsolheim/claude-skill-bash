# SKILL.md skeleton

Copy this skeleton when building a new skill from PDF(s). Replace `<bracketed>` placeholders. Sections marked OPTIONAL can be dropped for small skills.

The total length should stay under 500 lines per Anthropic guidance. Move chapter-level detail to chapter files; SKILL.md is the navigation and quick-lookup layer.

---

```markdown
---
description: <One paragraph in the user's query language. Cover scope (what domain), key trigger phrases (synonyms, identifiers like model codes / policy numbers / member numbers), and major topics. Be dense — the combined description + when_to_use is capped at 1,536 characters and competes for budget with other skills.>
---

# <Skill title in target user's language or source language — your choice based on what's natural>

<One or two sentences describing what authoritative knowledge this skill owns. State the scope, not the process for using it. E.g. "Authoritative data for X, Y, and Z from <source>(s)."  Skip if the description already says this clearly.>

## Navigation

| Topic | File |
|-------|------|
| <Short description of chapter 1 or section 1> | [chapters/01-<slug>.md](chapters/01-<slug>.md) |
| ... | ... |
| Original source PDF | [<source>.pdf](<source>.pdf) |

(For skills with many chapters, group by theme: "### Specifications", "### Operation", "### Maintenance", each with its own sub-table.)

## Quick-lookup

The most-asked facts. Put them here so a single-line answer doesn't require loading a full chapter.

### <Category 1, e.g. "Identifiers">

| Field | Value |
|-------|-------|
| <Key identifier name> | **<value>** |
| ... | ... |

### <Category 2, e.g. "Specifications">

| Parameter | Value |
|-----------|-------|
| <spec name> | <value with units> |
| ... | ... |

### <Category 3, e.g. "Contacts">

| Purpose | Contact |
|---------|---------|
| <e.g. emergency> | **<number>** |
| ... | ... |

## Common questions → file

| Question | Go to |
|----------|-------|
| <Likely question 1> | <Inline answer if very short, else> [chapter/section](path) |
| <Likely question 2> | ... |

(Populate this table from real or anticipated queries. Grow it over time when you spot questions that should have been here.)

## Diagram overview (OPTIONAL — only if skill has bundled diagrams)

| What | File |
|------|------|
| <description of figure> | [diagrams/<name>.png](diagrams/<name>.png) |
| ... | ... |

## Important notes (OPTIONAL — caveats, limitations, references)

- <Note about source date if web-fetched>
- <Note about which variant/model the data applies to>
- <Cross-reference to other skills if relevant>
```

---

## Worked example: motor manual skill

For a Yamaha F100 outboard motor manual covering three model variants:

```markdown
---
description: Yamaha F80B/F80C/F100D outboard motor (4-stroke) — owner's manual. Use for questions about motor specifications, oil capacity, spark plugs, fuel, propeller, fuses, maintenance intervals, warning lights, troubleshooting, motor start/stop, trim/tilt, winter storage, emergency procedures (water separator, manual start, fuse replacement), or other technical details about a Yamaha F100 outboard.
---

# Yamaha F80B/F80C/F100D Owner's Manual

**Models covered:** F80B (F80BET), F80C (F80CED), F100D (F100DET)
**Source:** `manual.pdf` (100 pages, 1st Edition March 2010)

## Navigation

| Chapter | Topic | File |
|---------|-------|------|
| 1 | Safety information | [chapters/01-safety-information.md](chapters/01-safety-information.md) |
| ... | ... | ... |

## Quick-lookup

### Critical specifications (F100D)

| Parameter | Value |
|-----------|-------|
| Max output | 73.6 kW (100 HP) @ 5500 rpm |
| Idle (neutral) | 700 ±50 rpm |
| Displacement | 1596 cm³ |
| Spark plug | NGK LFR5A-11, gap 1.0–1.1 mm |
| Engine oil capacity | 4.3 L total (3.5 L change w/o filter, 3.7 L w/ filter) |
| Gear oil F100D/F80B | 0.670 L |
| Min battery (CCA/EN) | 430 A |

### Tightening torques

| Component | Nm |
|-----------|-----|
| Spark plug | 25.0 |
| Propeller nut F100D | 35.0 |
| Engine oil drain bolt | 28.0 |

### Fuses

| Circuit | Amp |
|---------|-----|
| Starter relay | 30 A |
| Main switch / trim switch | 20 A |
| ECU / ignition / fuel pump / injector | 20 A |

## Common questions → file

| Question | Go to |
|----------|-------|
| How much oil does the engine take? | Quick-lookup or [chapters/09-maintenance.md](chapters/09-maintenance.md) |
| Which spark plug? | Quick-lookup — NGK LFR5A-11 |
| How to change engine oil? | [chapters/09-maintenance.md](chapters/09-maintenance.md) |
| Engine won't start — what do I check? | [chapters/10-trouble-recovery.md](chapters/10-trouble-recovery.md) |
| Manual start if starter fails? | [chapters/10-trouble-recovery.md](chapters/10-trouble-recovery.md) + [diagrams/emergency-starter-rope-1.png](diagrams/emergency-starter-rope-1.png) |
```

## Pitfalls to avoid in SKILL.md

- **Mixing source-of-truth with derived data.** If a fact lives elsewhere (e.g. in a chapter file, in another skill, on the web), point to it rather than duplicating. Duplication causes drift.
- **Describing how to maintain the skill in the skill itself.** Maintenance docs belong in CLAUDE.md or a separate README — not in SKILL.md, which is loaded into every triggered session and competes for context budget.
- **Knowing about presentation layers.** A skill should not say "this content is included in <other document>". The skill owns its data; what consumes it is not its concern.
- **Marketing-style headers.** Use simple, direct headings ("Specifications", "Maintenance"), not promotional ones ("Everything You Need to Know About Your Motor").
- **Too many synonyms in description.** Aim for the natural query terms — "polise" + "policy" + "forsikring" + "kasko" is fine, but listing every possible word will dilute matching.
