# Agent Operation Protocol (AOP)

[![AOP v1.0](https://img.shields.io/badge/AOP-v1.0-blue?style=flat-square)](https://agentoperationprotocol.org/v1.0)
[![Status: Draft](https://img.shields.io/badge/status-draft-yellow?style=flat-square)](https://agentoperationprotocol.org/v1.0)
[![License: CC BY 4.0](https://img.shields.io/badge/license-CC%20BY%204.0-lightgrey?style=flat-square)](https://creativecommons.org/licenses/by/4.0/)

**AOP is an open behavioral protocol for actors operating in shared operational graphs.**

It defines six laws that any conformant actor — human, AI agent, automation system, or external integration — must obey when performing actions against a governed graph system. AOP is technology-agnostic: no assumptions about implementation language, runtime, or graph engine.

## The Six Laws

1. **Read Before Write** — An actor must read current state before mutating it.
2. **Validate Schema Before Mutation** — All mutations must conform to declared schema.
3. **Preserve Graph Integrity** — No orphaned entities, dangling relations, or illegal cycles.
4. **Emit Events for Every Action** — Every mutation emits an event. No silent writes.
5. **Act Within Declared Capabilities** — Capability checks at action time, not session start.
6. **Respect Lifecycle Gates** — State transitions require declared capability.

## Specification

| Version | Date | Status | URL |
|---------|------|--------|-----|
| 1.0 | 2026-03-12 | Draft | [/v1.0/spec.md](./v1.0/spec.md) |

## Claiming Compliance

A system claiming AOP conformance should:

1. Reference this document and version (AOP v1.0) in its architecture documentation.
2. Document how each of the six laws is enforced at its API boundary.
3. Document its rejection response schema aligned with AOP rejection semantics.
4. Identify the audit log mechanism used to record governance rejections.

Add this badge to your project:

```markdown
[![AOP v1.0 Compliant](https://img.shields.io/badge/AOP-v1.0%20compliant-blue?style=flat-square)](https://agentoperationprotocol.org/v1.0)
```

## Reference Implementation

[OKL (Operational Kernel Layer)](https://github.com/ideacrafterslabs/okl) is the reference implementation of AOP v1.0.

## License

AOP is published under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You are free to implement, extend, and reference AOP in any project, commercial or otherwise, with attribution.

## Contributing

AOP is an open standard. Proposals to extend or amend the protocol are welcome via GitHub Issues and Discussions.
