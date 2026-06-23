![Holistic Code Specification](./assets/hcs-logo-landscape.png)

# Welcome to the Official Holistic Code Specification (HCS) Interpreter's Guide

This is the home of the complete Holistic Code Specification (HCS) reference documentation aimed at interpretation roles.

These documents provide all the information needed to understand and interpret HCS files — the structured-English, evidence-linked, pattern-compressed system blueprints generated from source code.

## Document Index

| Document | Purpose |
|----------|---------|
| [Overview of HCS](./01-hcs-overview.md) | What HCS is, its purpose, core concepts, and holistic dimensions |
| [Language Reference](./02-hcs-language-reference.md) | Grammar, syntax, keywords, casing rules, and statement schemas |
| [Intermediate Model](./03-hcs-intermediate-model.md) | Machine-facing data model, types, and evidence structures |
| [HCS Fact Types](./04-hcs-fact-types.md) | Complete taxonomy of fact types captured from source code |
| [HCS Patterns](./05-hcs-patterns.md) | Pattern system, type system, expansion templates, and standard library |
| [Fact ABI JSON Schema](./hcs-fact-abi.schema.json) | Authoritative draft 2020-12 schema for the v0.5 behavioural/HXL fact types (FR-SCH-1) |
| [Migration v0.4 → v0.5](./MIGRATION-v0.4-to-v0.5.md) | Breaking confidence-vocabulary change + alias map; dual versions; HXL/behavioural updates |
| [Changelog](./CHANGELOG.md) | Version history and release notes |

## Reading Order

For comprehensive understanding:
1. Start with **Overview of HCS** to understand what HCS is and why it exists
2. Read **Language Reference** to understand the DSL syntax
3. Study **Intermediate Model** to understand the underlying data structures
4. Reference **HCS Fact Types** for the evidence taxonomy
5. Consult **HCS Patterns** for pattern compression and expansion

## Key Concepts

### HCS is NOT documentation
HCS is a **specification artifact** with:
- Structured English statements
- Typed semantics (every statement maps to a machine-checked schema)
- Traceability (every claim links back to evidence in code)
- Pattern compression (common idioms collapse into named patterns)

### Behavioural Equivalence (Fact ABI v0.3–v0.4)
Beyond structure, HCS pins *what the system does*: **behavioural assertions** bound
to the facts they constrain, and decision logic captured in **HXL** — the
deterministic, language-agnostic HCS Expression Language. Two honest, separate
measures report progress: `behaviouralConfidence` (surface coverage) and
`logicCoverage` (how much decision logic was captured faithfully). No LLM is ever
in the path. See [Overview · Behavioural Equivalence](./01-hcs-overview.md#behavioural-equivalence--a-cross-dimension-proof-layer).

### Reproducibility
A system is reproducible from HCS when it contains enough detail to regenerate:
- Domain behaviour (use-cases and workflows)
- Interface contracts (UI + APIs + message schemas)
- Data structures (entities, constraints, relationships)
- Security posture (auth flows, authorization rules, boundaries)
- Integration behaviour (calls, retries, timeouts, idempotency)

---

**© 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
