# Agent Operation Protocol (AOP)
Version: 1.0
Status: Draft
Date: 2026-03-12

---

## Overview

AOP is a behavioral protocol for actors performing operations against a
shared operational graph. It defines six laws that any conformant actor —
human, AI agent, automation system, or external integration — must obey
when performing actions against a governed graph system.

AOP is technology-agnostic. It makes no assumptions about implementation
language, runtime, or graph engine. Any system that stores entities, emits
events, and enforces actor permissions can adopt AOP compliance.

---

## Changelog

| Version | Date       | Changes          |
|---------|------------|------------------|
| 1.0     | 2026-03-12 | Initial draft    |

---

## Scope

AOP governs **actors performing actions** — mutations and queries — against
a shared operational graph. An actor is any identity capable of performing
an action: a human user, an AI agent, an adapter, or an automated system.

AOP does not govern:
- Internal graph engine implementation
- Transport or networking protocols
- Identity provider configuration
- Storage or persistence mechanisms

---

## The Six Laws

### Law 1 — Read Before Write

An actor must read the current state of an entity before mutating it.
Blind writes are rejected.

**Rationale:** Prevents race conditions. Ensures actors operate on accurate
state, not stale assumptions.

**Conformance:** The system must track whether the requesting actor has read
the entity within the current operation context. A write submitted without
a prior read is rejected before application.

---

### Law 2 — Validate Schema Before Mutation

All mutations must conform to the entity's declared schema. Type mismatches,
missing required attributes, and invalid relation targets are rejected before
the mutation is applied.

**Rationale:** Preserves graph integrity. Prevents malformed data from
propagating through downstream subscribers.

**Conformance:** Schema validation occurs at the API boundary. A mutation
that fails schema validation is rejected with a structured error identifying
the failing field and expected type. The entity is not modified.

---

### Law 3 — Preserve Graph Integrity

An actor may not create orphaned entities, dangling relations, or circular
dependencies that violate the graph's declared constraints.

Deletions cascade according to defined rules; they do not leave broken
references.

**Rationale:** A graph with dangling references or cycles is unreliable for
traversal, audit, and downstream event consumption.

**Conformance:** The system must validate all relation targets exist before
creating a relation. Deletion operations must apply cascade rules atomically.
Circular dependency checks must run on relation creation where cycles are
prohibited by schema.

---

### Law 4 — Emit Events for Every Action

Every mutation emits an event. There are no silent writes. An actor cannot
opt out of event emission.

**Rationale:** Events are the foundation of observability, auditability, and
integration. Silent mutations cannot be audited, replayed, or subscribed to
by downstream systems.

**Conformance:** The system must emit a structured event for every accepted
mutation before returning a success response to the caller. The event must
identify: the actor, the entity, the action type, and the timestamp. Partial
emission (event emitted for some mutations but not others) is non-conformant.

---

### Law 5 — Act Within Declared Capabilities

An actor may only perform actions permitted by its current capabilities and
permission level. Capability checks are performed at action time, not at
session start. Capabilities may be revoked between requests.

**Rationale:** Least-privilege enforcement. An actor's authority must reflect
its current grant, not its authority at login time.

**Conformance:** The system must check capabilities at the moment of each
action, not cache them from session initialization. A capability revoked
between two requests must take effect on the next request. Rejections must
identify the missing capability specifically (see Rejection Semantics below).

---

### Law 6 — Respect Lifecycle Gates

An actor may not move an entity to a lifecycle state it does not have the
capability to reach. State transitions are unidirectional unless explicitly
defined as reversible in the schema.

**Rationale:** Lifecycle gates enforce workflow integrity. An entity in
`review` cannot be moved to `published` by an actor without publish authority,
regardless of their other capabilities.

**Conformance:** The system must maintain a declared state machine per entity
type. Each transition must declare a required capability. Attempting a
transition without the required capability is rejected. Undeclared transitions
(states not connected in the machine) are always rejected.

---

## Rejection Semantics

A conformant AOP system must make rejections **typed, structured, and
queryable**. Silent rejections are non-conformant.

Every rejection response must include:

| Field             | Description                                              |
|-------------------|----------------------------------------------------------|
| `rejection_type`  | Class of check that failed: `permission`, `capability`,  |
|                   | `lifecycle_gate`, `aop_law`, `schema`, `not_found`,      |
|                   | `conflict`                                               |
| `check`           | The specific check that failed (e.g. `power.approve`)    |
| `required`        | What was required to pass the check                      |
| `actor_had`       | What the actor actually held; null if nothing relevant   |

Rejections caused by `permission`, `capability`, `lifecycle_gate`, or
`aop_law` failures must be recorded in the system's audit log, associated
with the actor and the entity. Rejections caused by `schema`, `not_found`,
`conflict`, or `rate_limit` are surfaced in the API response only — they
are not governance events.

**Silent rejection is non-conformant.** A system that discards a rejected
action without emitting a structured response and (where required) an audit
log entry violates AOP.

---

## Compliance Levels

A system claiming AOP compliance must implement all six laws and the
rejection semantics above. There are no partial compliance tiers — the
protocol is a complete behavioral contract.

Systems may extend AOP with additional laws (e.g. domain-specific mutation
constraints), provided the six core laws are not weakened.

---

## Claiming Compliance

A system claiming AOP conformance should:

1. Reference this document and version (AOP v1.0) in its architecture
   documentation.
2. Document how each of the six laws is enforced at its API boundary.
3. Document its rejection response schema and confirm alignment with the
   rejection semantics above.
4. Identify the audit log mechanism used to record governance rejections.

---

## Reference Implementation

OKL (Operational Kernel Layer) is the reference implementation of AOP v1.0.
Architecture: `docs/okl-architecture.md`

---

End of Document
