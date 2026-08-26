---
name: general-architecture-principles
description: General software architecture principles — extending the existing domain model instead of creating parallel per-feature vocabularies, separating generic capability from feature-specific judgment, naming from the ubiquitous language, and ranking model coherence above change isolation. Use when designing, architecting, or reviewing features, APIs, services, or data models — any time new types, endpoints, resources, or concepts are about to be introduced into an existing system.
metadata:
  version: 1.0.0
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

## Plans present the model, not just the mechanics

- Every design or plan carries a first-class section: **what this builds on** (the existing
  concepts reused or extended) and **what is genuinely new, and why it must be**.
- Contracts, file lists, and test plans describe mechanics; the reviewer must be able to
  review the *model decision* directly, without reverse-engineering it from JSON examples.
- If the builds-on section is empty, the design is almost certainly wrong — or the system is
  brand new.
