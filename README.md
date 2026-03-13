# Agent Operation Protocol (AOP)

[![AOP v0.5](https://img.shields.io/badge/AOP-v0.5-blue?style=flat-square)](https://agentoperationprotocol.org/v0.5)
[![Status: Draft](https://img.shields.io/badge/status-draft-yellow?style=flat-square)](https://agentoperationprotocol.org/v0.5)
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
| 0.5 | 2026-03-12 | Draft | [/v0.5/spec.md](./v0.5/spec.md) |

## Claiming Compliance

A system claiming AOP conformance should:

1. Reference this document and version (AOP v0.5) in its architecture documentation.
2. Document how each of the six laws is enforced at its API boundary.
3. Document its rejection response schema aligned with AOP rejection semantics.
4. Identify the audit log mechanism used to record governance rejections.

Add this badge to your project:

```markdown
[![AOP v0.5 Compliant](https://img.shields.io/badge/AOP-v0.5%20compliant-blue?style=flat-square)](https://agentoperationprotocol.org/v0.5)
```

## Relation to Adjacent Standards

AOP is the behavioral governance layer. It sits above transport and below application logic — complementary to these standards, not competing.

| Standard | Role | Relationship to AOP |
|---|---|---|
| **MCP** (Model Context Protocol) | Transport and tool-calling layer | AOP governs how actors behave once connected; MCP governs how they connect and invoke tools. Use both. |
| **A2A** (Agent-to-Agent Protocol) | Agent coordination and messaging | A2A defines how agents communicate with each other; AOP defines the behavioral contract each agent must honor when acting on a shared system. |
| **CloudEvents** | Event schema and envelope standard | AOP's Law 4 requires structured event emission. AOP events are compatible with CloudEvents but do not require it — any structured event format satisfies the law. |
| **OAuth / OpenID Connect** | Identity and authorization | AOP assumes identity is already established (out of scope). Law 5 requires capability checks at action time; the underlying grant mechanism (OAuth scopes, OIDC claims, etc.) is the implementor's choice. |

## Reference Implementation

We're building a reference implementation. Not ready to name it yet — but the protocol is open and ready for feedback now.

## License

AOP is published under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You are free to implement, extend, and reference AOP in any project, commercial or otherwise, with attribution.

## Contributing

AOP is an open standard. Proposals to extend or amend the protocol are welcome via GitHub Issues and Discussions.
