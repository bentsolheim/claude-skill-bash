---
name: rest-api-principles
description: REST API design principles — resource modeling, URL structure, versioning, collections, sub-collections, ownership, operations, reports, methods and status codes. Use when designing, adding, or reviewing REST API endpoints, or deciding how a domain concept maps to resources and URLs.
metadata:
  version: 0.9.0
---

# REST API Principles

## Resources

- Every core domain concept is its own resource: `/api/v{version}/{entities}/{id}`. Resource names are plural kebab-case, e.g. `/api/v1/flow-categories/{id}`.
- When the domain language treats a multi-word concept as a single word, write it as one word. Document such exceptions to the naming rule explicitly in the project applying it.

## Versioning

- The `v{version}` segment is the major version only: `v1`, never `v1.2`.
- It is required for public APIs, where we do not control all clients; optional for internal APIs, where all clients can easily be moved along with API changes.

## Methods and status codes

- `GET` is safe; `PUT` and `DELETE` are idempotent; `POST` creates or executes.
- `400 Bad Request` for validation failures; `409 Conflict` for requests that conflict with the resource's current state.

## Collections

- The resource root `/api/v{version}/{entities}` is a collection listing all resources of that type.
- If an unfiltered listing is inappropriate, the collection takes query parameters to filter; an unfiltered request returns `400 Bad Request`, and the error names the required filters.
- There is no pagination convention — deliberately, not as an oversight. Introduce one only when scale demands it.

## Ownership and nesting

- If an entity owns a collection of sub-resources (deleting the parent deletes them), represent it as a sub-collection under the parent: `/api/v{version}/{parents}/{id}/{children}`.
- Cap owned sub-collections at one level under the parent unless there is a documented reason to go deeper.
- If an entity holds a collection of entities it does not own (independent lifecycle), those entities are their own root collection. In the referencing entity's representation they appear only as **refs** — the minimal representation of a resource: its id, plus a display name where appropriate.

## Operations

- An operation on the domain model is its own resource, suffixed `-operation`, e.g. `/api/v1/migrate-flow-category-operations`.
- Operation names are verb-first and keep the `-operation` suffix even where a noun name (e.g. `/flow-category-migrations`) would read more naturally — self-declaring and greppable wins.
- Operations may likewise be organized under an `/operations` sub-resource, and deeper, when it makes sense.
- An operation may span several domain objects, of the same or of different types, when appropriate.
- An operation has its own id and is referenceable by URL when its execution must be referenced afterwards (async status, audit/history); otherwise it executes without one.
- `POST` to the operation resource executes it, creating an execution at `/{id}` when the operation has ids.
- A long-running operation with an id returns `202 Accepted` with a `Location` header pointing at the execution; its status is read from that URL.
- An operation has its own response, with the relevant resources embedded or linked.

## Reports

- A request that returns data from multiple entities/resources is its own top-level resource, suffixed `-report`, e.g. `/api/v1/flow-instance-data-report`.
- Report names are **singular**, unlike collections: a report is a parameterized singleton view — nothing is enumerated and there is no `/{id}`. If a report needs persisted, addressable runs, model it as an operation with ids instead.
- Reports may be organized under a `/reports` sub-resource, and deeper when needed, e.g. `/api/v1/reports/flow-details/process-report?…`. The grouping prefix stays plural; the leaf names one report.
- A report is read-only: `GET`, with query parameters selecting its content. A request missing required parameters returns `400 Bad Request` naming them.
- A report resource may contain redundant data — values copied from other resources or entities — to be concise and efficient.
- That does not displace embedding or referencing related entities where it makes sense: if nearly all of a resource's properties would be copied into the report, consider embedding the resource instead.
