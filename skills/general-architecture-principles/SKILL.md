---
name: general-architecture-principles
description: General software architecture principles — extending the existing domain model instead of creating parallel per-feature vocabularies, separating generic capability from feature-specific judgment, naming from the ubiquitous language, and ranking model coherence above change isolation. Use when designing, architecting, or reviewing features, APIs, services, or data models — any time new types, endpoints, resources, or concepts are about to be introduced into an existing system.
metadata:
  version: 1.2.0
---

# General Architecture Principles

The recurring failure mode these principles guard against: each feature designed as its own
self-contained unit — its own DTOs, its own endpoint namespace, its own vocabulary — because that
is locally clean and low-risk. Each such feature works; the system accretes disjointed,
intermixed concepts with diffuse boundaries and a redundant overall model. Design for the
system, not the feature.

## Model the system, not the feature

- Every design starts from the model the system already has, not from the feature's needs.
- Before introducing **any** new type, endpoint, projection, or concept: enumerate the existing
  concepts that overlap it. Reuse or extend is the default.
- A new concept must **earn its existence**: an explicit, written justification — in the plan,
  before code — of why no existing concept can carry it. "It keeps the change self-contained"
  is not a justification; it is the failure mode.
- When an existing concept almost fits, prefer evolving it (and its other consumers) over
  cloning it. The clone is cheaper today and more expensive every day after.

## Generalize siblings — don't mint a twin

- Before naming a new domain concept, search the whole domain for the same concept already
  living under a narrower, subdomain-scoped name — a per-subdomain exception, status, guard, or
  policy that expresses the identical idea for its corner of the domain.
- The sibling will not be found by name — by definition it is named for its subdomain. Search by
  mechanism and behavior instead: what the codebase already throws, returns, or checks in the
  analogous situation (base types, status-code mappings, naming suffixes, catch sites).
- Finding one is the signal to **devise the general concept**: lift the existing narrow one to
  the domain level and let both subdomains speak it — never mint a second narrow equivalent
  beside it. Two subdomain twins are worse than either alone: they prove the domain has the
  concept while denying it a name, and every future subdomain will mint a third.
- The moment of the second instance is the cheapest time to generalize: the two concrete use
  cases in hand are exactly the evidence needed to name and shape the general concept correctly.

## Separate capability from judgment

- When building an operation, tool, or workflow, split what it *knows* from what it *decides*:
  the reusable read model (what any feature would want to know about these entities) is one
  resource; the operation-specific verdict (eligibility, blockers, what-would-happen) is
  another.
- Test every field of a response against: "which other feature would want this, unchanged?"
  If the answer is none, the field belongs to the operation — not to the entity view it is
  riding on. If the answer is many, it must not be trapped inside an operation-specific
  contract.
- A fused capability-plus-judgment resource serves exactly one consumer forever; split
  resources compound.

## Vocabulary is the model

- Systems have a ubiquitous language: the names their domain already speaks. New code speaks
  that language.
- A design that introduces names absent from the existing domain vocabulary is showing a
  design smell — stop at each new name and check whether an established concept already means
  this. Parallel vocabularies (a feature's private `Ref`, `Row`, `Entry`, `Info` shadowing
  real domain types) are how two models end up describing one domain.
- Naming conventions are part of the language — including its documented exceptions. Follow
  both; never invent a third style.

## Coherence outranks isolation

- Small-diff, low-blast-radius, nothing-disturbed is the natural reflex ranking. It is wrong
  as a top value: it optimizes each change at the expense of the system.
- Prefer landing on the shared concept even when it widens the change — touching the shared
  thing plus its consumers is usually the cheaper choice over the system's lifetime.
- Isolation is still a virtue *within* the right model. It stops being a virtue when it is the
  reason a parallel model exists.

## Layers wrap — don't reach past them

- In a layered architecture, when a service object clearly has the responsibility to wrap a data
  access object, the data access object is not used directly from elsewhere. Go through the
  service object, and **extend it** when a data-access feature it should expose is missing —
  that extension is the service layer doing its job, not bloat.
- Reaching past the owning layer splits ownership: invariants, priming, caching, and access
  rules the service enforces silently do not apply on the bypass path, and the next reader can
  no longer trust the service as the single account of how its data is touched.
- A direct data-access dependency outside its owning service is acceptable only when genuinely
  necessary — and then the injection site says why.
- The discipline generalizes to every boundary, not just service→data: whatever component wraps
  a subsystem *is* that subsystem's access path.

## Plans present the model, not just the mechanics

- Every design or plan carries a first-class section: **what this builds on** (the existing
  concepts reused or extended) and **what is genuinely new, and why it must be**.
- Contracts, file lists, and test plans describe mechanics; the reviewer must be able to
  review the *model decision* directly, without reverse-engineering it from JSON examples.
- If the builds-on section is empty, the design is almost certainly wrong — or the system is
  brand new.
