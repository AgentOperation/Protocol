# Contributing to AOP

AOP is an open protocol draft. Community input is genuinely welcome — the spec is better
when people who implement it have a voice in its evolution.

---

## How to propose an amendment

| Change type | Start here |
|-------------|-----------|
| Question / early idea | GitHub Discussion |
| Bug in existing text (ambiguity, contradiction, errata) | GitHub Issue |
| Concrete text change | GitHub PR (link to or include the issue it resolves) |

For anything beyond a trivial editorial fix, open an issue first so the idea can be
discussed before you invest time drafting spec language.

---

## What makes a good proposal

A well-formed proposal answers four questions:

1. **Problem** — what is ambiguous, missing, or wrong in the current spec?
   Quote the relevant section if applicable.
2. **Proposed change** — exact text change or addition (diff or prose).
3. **Rationale** — why this makes the protocol more correct, clearer, or more
   useful for conformant systems.
4. **Impact on conformant systems** — does this break existing compliant
   implementations? If yes, how significant is the migration cost?

Proposals that skip the impact analysis are harder to evaluate and may sit longer.

---

## How decisions are made

AOP v0.5 is maintainer-driven with community input. Concretely:

- Minor clarifications: maintainer discretion, merged after a brief review window.
- Substantive changes (new normative text, changed semantics): minimum **14-day**
  open comment period on the issue/PR before merge.
- Breaking changes (see CHANGELOG versioning policy): broader discussion expected;
  maintainer summarizes disposition of all substantive comments before merging.

This process is intentionally lightweight for a v0.5 draft. It will be revisited
when the spec reaches Candidate status.

---

## Path from proposal to spec change

```
Issue / Discussion
  └─ maintainer acknowledges + labels (errata | additive | breaking)
       └─ comment period (14 days for substantive; shorter for errata)
            └─ PR with spec text + CHANGELOG entry
                 └─ review + merge → version increment
```

After merge, the CHANGELOG records the change and its disposition.

---

## Style guide (spec text)

- Use RFC 2119 keywords (MUST, SHOULD, MAY) for normative statements.
- Keep sentences short; one normative obligation per sentence.
- Informative text goes in notes or annexes, not the law bodies.
- Line length: 100 chars max.
