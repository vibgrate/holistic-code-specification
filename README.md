![Holistic Code Specification](./assets/hcs-logo-landscape.png)

# Welcome to the Official Holistic Code Specification (HCS) Interpreter's Guide

This is the home of the complete Holistic Code Specification (HCS) reference documentation aimed at interpretation roles.

These documents provide all the information needed to understand and interpret HCS files — the structured-English, evidence-linked, pattern-compressed descriptions of a system generated from its source code.

> **Current release: HCS v0.5.** The [Changelog](./CHANGELOG.md) is the single source of truth for the current version.

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

### Behaviour pinning
Beyond structure, HCS captures *what the system does*: **behavioural assertions**
bound to the facts they constrain, with the decision logic inside methods lowered
into **HXL** — the deterministic, language-agnostic HCS Expression Language. Two
honest, separate measures report how much was captured — `behaviouralConfidence`
(the share of behaviour-bearing surfaces pinned by an assertion) and
`logicCoverage` (how much decision logic was captured faithfully vs. left
explicitly opaque). Every assertion is evidence-linked and **no LLM is ever in the
path**. See [Overview · Behavioural layer](./01-hcs-overview.md#behavioural-layer--pinning-what-a-system-does).

## What HCS Delivers

HCS exists to **discover and describe what a system actually does** — its
behaviour, its data, and its place in the wider ecosystem — as deterministic,
evidence-linked facts. The capabilities we assert and deliver today:

- **System discovery & description** — the API surface and routes, the data model
  (entities, fields, relationships, constraints), security boundaries and rules,
  and the business logic inside methods, each linked back to the exact source
  evidence it came from.
- **Ecosystem & dependency mapping** — the external services, integrations,
  events, message schemas, and data stores a system depends on, so you can see how
  it connects to everything around it.
- **Behaviour pinning & drift detection** — coverage-honest, diffable assertions
  about observable behaviour that flag when a system's behaviour changes between
  scans.
- **Migration-grade inventory** — a faithful, traceable catalogue of a system to
  scope, de-risk, and plan a modernization.

Together these power the Vibgrate dashboard: **drill into any system and describe
it in detail** — what it does, what it depends on, and how it behaves — all
traceable to evidence in the source.

> **On reproduction.** HCS makes no claim that a system can be regenerated from
> its specification. Full behavioural reproduction is a research direction we
> track openly via the reproduction harness — it is *not* a current guarantee, and
> nothing here should be read as one.

---

**© 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
