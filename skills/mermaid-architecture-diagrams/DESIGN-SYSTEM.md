# Design system for architecture diagrams

Rules that make a diagram *set* read as one coherent artifact. Adapt the specifics per project; never abandon the principles.

## The 5-second test

A reader must grasp the main story in ~5 seconds: what talks to what, through what, and where the boundaries are. Details (ports, versions, instance sizes) are texture, not story. If the first impression is spaghetti, redesign — don't decorate.

## Semantic color tokens

Color must *mean* something, and the same thing everywhere. Default token set (Okabe-Ito-derived, colorblind-safe, verified in both major pipelines):

| Token | Meaning | Fill | Border |
|-------|---------|------|--------|
| `compute` | Things you run (services, hosts, load balancers, CI) | `#E3EEF6` | `#0072B2` |
| `datastore` | Data at rest (databases, buckets, backup stores) | `#FBEEDB` | `#E69F00` |
| `external` | Third-party SaaS / systems outside your control | `#F2F2F2` | `#8C8C8C` (dashed) |
| `actor` | People and client systems | `#E2F3ED` | `#009E73` |
| `gap` / attention | Known risks, gaps, warnings | keep token fill | `#D55E00`, 2.5px border accent — never a red fill |
| boundary | Real containment only (cloud region, network zone, host) | `#F7F7F5` / `#FAFAF8` | `#BBBBBB` |

Rules of use:

- **≤2 strong hues per diagram** plus neutrals; reserve one accent (the vermillion above) exclusively for "look here" (gaps, alerts).
- Projects may re-map hues (e.g. to brand colors) but must keep *one meaning per color* across the whole diagram set, and should keep contrast ≥ AA for text (`#1A1A1A` on light fills).
- Never rely on Mermaid theme defaults for node colors — they differ between engine versions. Define complete classDefs (see TECHNIQUES.md).

## Edge grammar

Line style is semantic, set-wide:

- **Solid** — runtime traffic / primary flow (`https :443`, `:5432`)
- **Dashed** — automation and background flows (scheduled jobs, image pulls, scrapes, backups)
- **Dashed + accent color** — alert/notification delivery (if used, use it in every diagram of the set)
- Arrowheads point with the payload; the label names the protocol/port or the action (`ssh :22`, `git push`). Terse labels; edge labels get a white background (set `edgeLabelBackground`).

## Layout principles

1. **Direction per purpose**: containment/topology TB, pipelines LR — but prefer TB when the destination is print (LR strips shrink to unreadable type at text width).
2. **Crossing budget**: ≤2–3 edge crossings per diagram, zero is achievable more often than you think. Control it with declaration order and invisible `~~~` links; consider routing an edge to a *boundary* instead of multiple inner nodes (one edge to the security-group box ≫ two crossing edges to its hosts).
3. **Subgraphs are real boundaries only** (region, network zone, host, instance). Max 3 nesting levels. If a box is just a visual grouping with no containment meaning, delete it.
4. **One landing depth per concept**: edges representing the same kind of flow should terminate at the same depth (all clients on the node, not one on the node and one on its enclosing box).
5. **Zoning**: place actors/sources in one band (top or left), your system in the middle, external systems in an outer band. Declaration order drives placement.

## Labels

- Node: short title (≤3 words) + at most one or two `<small>` detail lines. Paragraphs in boxes kill diagrams.
- Subgraph titles: ≤24 characters, single line (longer wraps/clips — verified in both pipelines).
- Break long detail lines explicitly with `<br/>` — auto-wrap breaks mid-token at ~200px.
- When detail doesn't fit, **delegate to the figure caption** and write the caption text alongside the diagram source. A caption-carried legend beats an in-diagram legend when space is tight.

## Set consistency

All diagrams in a set share: identical init block, identical classDef tokens, identical edge grammar, one naming scheme per entity (a host named "Prod server 2" in one diagram is not "Prod host" in the next), and one legend treatment. Inconsistency between sibling diagrams reads as sloppiness even when each diagram is individually fine.

## Aesthetic levers (verified)

Beyond the flat baseline, these render in both major pipelines — apply with restraint and consistently:

- **Rounded corners**: `rx:10,ry:10` in classDef (and in subgraph `style` lines)
- **Translucency**: `fill-opacity:0.12` with a solid border reads as a tint
- **Weight**: `font-weight:bold` for key nodes; `stroke-width` for emphasis (pick one width, deviate only for accents)
- **Shadows**: only via init `themeCSS` (e.g. `.node rect { filter: drop-shadow(2px 3px 4px rgba(0,0,0,0.25)); }`) — never in classDef (parse error in newer Mermaid). Tune them strong enough to survive print scaling; expect clipping at canvas edges.
