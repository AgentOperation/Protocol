# Agent Operation Protocol (AOP)
Version: 0.5
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
| 0.5     | 2026-03-12 | Initial public draft    |

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

## Glossary

**Actor** — Any identity capable of performing an action against an operational
graph; includes human users, AI agents, adapters, and automated systems.

**Operational graph** — A governed, structured set of entities and relations
against which actors perform reads, mutations, and lifecycle transitions.

**Lifecycle gate** — A declared constraint that restricts which capability an
actor must hold in order to move an entity from one lifecycle state to another.

**Capability** — A discrete, named grant of authority that permits an actor to
perform a specific class of action; capabilities are checked at action time and
may be revoked between requests.

**Mutation** — Any action that changes the state of an entity in the operational
graph, including creates, updates, deletes, and lifecycle transitions.

**Rejection** — A structured, typed response returned when an action fails
conformance checking; rejections are never silent and, for governance failures,
are recorded in the audit log.

**Conformance** — The condition of a system or actor that satisfies all six AOP
laws and the rejection semantics defined in this document; conformance has no
partial tiers.

---

## Non-Goals

The following are explicitly outside AOP's scope. AOP does not govern, define,
or constrain any of these concerns.

**Transport protocol.** AOP does not specify how messages are carried between
actors and the system. HTTP, gRPC, WebSockets, message queues, and in-process
calls are all equally valid transport choices.

**Serialization format.** AOP does not require a specific wire format. JSON,
MessagePack, Protobuf, or any other encoding may be used provided the
structured rejection fields are representable.

**Agent discovery and registry.** AOP does not define how actors are discovered,
registered, or enumerated. Actor identity is assumed to be established before
any AOP-governed action is attempted.

**Authentication and identity.** AOP assumes that the calling actor's identity
has already been verified by an external mechanism. AOP enforces what an
authenticated actor may do; it does not perform authentication itself.

**Storage engine and persistence.** AOP does not specify how the operational
graph is stored, indexed, or replicated. Any persistence layer that can enforce
the six laws at its API boundary is acceptable.

**Event delivery guarantees.** AOP requires that events be emitted for every
mutation; it does not specify delivery semantics (at-most-once, at-least-once,
exactly-once) or the infrastructure used to carry those events.

**Conflict resolution strategy.** AOP requires that conflicts be surfaced as
typed rejections; it does not prescribe merge strategies, optimistic locking
algorithms, or CRDT policies.

**Inter-agent orchestration.** AOP governs individual actions at the API
boundary. It does not define how multiple actors coordinate, how workflows are
orchestrated across agents, or how consensus is reached between actors.

---

## Conformance Examples

Each example below demonstrates one conformant and one non-conformant scenario
per law, with the expected rejection response for the non-conformant case.
Examples use pseudocode and JSON. They are implementation-agnostic.

---

### Law 1 — Read Before Write

**Conformant**
```
actor.read("issue:42")           // read recorded in operation context
actor.update("issue:42", { status: "in-progress" })  // accepted
```

**Non-conformant**
```
actor.update("issue:42", { status: "in-progress" })  // no prior read
// → rejected
```

**Expected rejection**
```json
{
  "rejection_type": "aop_law",
  "check": "law1.read_before_write",
  "required": "entity must be read within current operation context before mutation",
  "actor_had": null
}
```

---

### Law 2 — Validate Schema Before Mutation

**Conformant**
```
actor.read("issue:42")
actor.update("issue:42", { priority: "high" })  // "priority" is a declared field; value matches enum
// → accepted
```

**Non-conformant**
```
actor.read("issue:42")
actor.update("issue:42", { priority: 9001 })  // "priority" expects enum string, not integer
// → rejected; entity not modified
```

**Expected rejection**
```json
{
  "rejection_type": "schema",
  "check": "field.type_mismatch",
  "required": "priority must be one of: low, medium, high, critical",
  "actor_had": "integer 9001"
}
```

---

### Law 3 — Preserve Graph Integrity

**Conformant**
```
actor.read("issue:42")
actor.create_relation("issue:42", "blocks", "issue:99")  // issue:99 exists
// → accepted
```

**Non-conformant**
```
actor.read("issue:42")
actor.create_relation("issue:42", "blocks", "issue:999")  // issue:999 does not exist
// → rejected
```

**Expected rejection**
```json
{
  "rejection_type": "not_found",
  "check": "graph.relation_target_exists",
  "required": "relation target issue:999 must exist before relation can be created",
  "actor_had": null
}
```

---

### Law 4 — Emit Events for Every Action

**Conformant**
```
actor.read("issue:42")
actor.update("issue:42", { status: "done" })
// system emits: { actor, entity: "issue:42", action: "update", timestamp }
// response returned to caller
```

**Non-conformant** (system behavior, not actor behavior)
```
// System applies mutation to the graph but returns response without emitting event.
// This is a system-level AOP violation.
```

**Expected audit outcome** — A conformance test must observe the event in the
event log before the success response is considered valid. A success response
with no corresponding event is a non-conformant system response.

```json
{
  "rejection_type": "aop_law",
  "check": "law4.event_emission_required",
  "required": "structured event must be emitted before success response is returned",
  "actor_had": null
}
```

---

### Law 5 — Act Within Declared Capabilities

**Conformant**
```
// actor holds capability: "issue.approve"
actor.read("issue:42")
actor.transition("issue:42", "approved")  // capability check passes
// → accepted
```

**Non-conformant**
```
// actor does NOT hold capability: "issue.approve"
actor.read("issue:42")
actor.transition("issue:42", "approved")
// → rejected
```

**Expected rejection**
```json
{
  "rejection_type": "capability",
  "check": "issue.approve",
  "required": "actor must hold capability issue.approve to perform this action",
  "actor_had": ["issue.read", "issue.comment"]
}
```

---

### Law 6 — Respect Lifecycle Gates

**Conformant**
```
// entity state: "draft"; actor holds capability: "issue.publish"
// declared transition: draft → published requires issue.publish
actor.read("issue:42")
actor.transition("issue:42", "published")
// → accepted
```

**Non-conformant (undeclared transition)**
```
// entity state: "published"
// no declared transition from published back to draft
actor.read("issue:42")
actor.transition("issue:42", "draft")
// → rejected; transition does not exist in state machine
```

**Expected rejection**
```json
{
  "rejection_type": "lifecycle_gate",
  "check": "transition.declared",
  "required": "transition from published to draft is not declared in the entity state machine",
  "actor_had": null
}
```

**Non-conformant (missing capability for valid transition)**
```
// entity state: "review"; actor lacks capability: "issue.publish"
// declared transition: review → published requires issue.publish
actor.read("issue:42")
actor.transition("issue:42", "published")
// → rejected
```

**Expected rejection**
```json
{
  "rejection_type": "lifecycle_gate",
  "check": "issue.publish",
  "required": "transition from review to published requires capability issue.publish",
  "actor_had": ["issue.read", "issue.edit"]
}
```

---

End of Document
