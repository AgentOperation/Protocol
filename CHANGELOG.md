# Changelog

## Versioning Policy

### Lifecycle stages

| Stage | Meaning |
|-------|---------|
| **Draft** | Under active development; may change without notice |
| **Candidate** | Feature-complete; community review period; breaking changes unlikely |
| **Stable** | Frozen; only additive changes or errata |

Versions use `MAJOR.MINOR` (no patch level for a spec).

### Breaking vs additive

**Breaking** — requires a new MAJOR version:
- Adding a new mandatory law
- Changing the normative text of an existing law in a way that invalidates previously-conformant systems
- Removing a law
- Changing the meaning of a core term (actor, graph, capability, lifecycle gate)

**Additive** — MINOR increment, no conformance break:
- Clarifying normative language without changing behavior
- Adding informative examples, guidance, or annexes
- Adding optional extension points
- Errata / editorial corrections

### Community input

1. Proposals arrive as GitHub Issues (discussion) or PRs (concrete text).
2. Significant changes collect a minimum 14-day comment period before promotion to Candidate.
3. Maintainer(s) incorporate feedback and note disposition in the issue/PR before merging.
4. CHANGELOG entry required for every MINOR or MAJOR increment.

---

## v0.5 — Draft (2026-03-12)

Initial public draft. Defines the six laws, conformance requirements, rejection
semantics, and compliance badge guidance.

**Scope:** behavioral laws for actors operating in shared operational graphs.
**Status:** open for community feedback; all sections subject to change.
