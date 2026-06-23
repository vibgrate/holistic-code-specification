# HCS v0.5 — Conformance hardening

**Targets:** Fact ABI v0.5 / Model Schema v0.5 · **Current release.**

v0.5 makes HCS objectively **conformance-testable**: every derived number is now a
deterministic function of facts in the model, the expression layer closes its
remaining gaps, and the trust/version vocabulary is unified. It also **sharpens the
claims**: HCS is positioned as a way to *discover and describe what a system does,
what it depends on, and how it behaves* — not as a system-reproduction tool.

## ⚠️ Breaking change — claim-confidence vocabulary

The per-fact `confidence` vocabulary is now exactly **`Observed | Derived |
Hypothesized`**. The v0.4 values are removed from the producer vocabulary; for the
whole v0.5 line consumers MUST accept the deprecated inputs and normalise them
(removed in v0.6):

| Deprecated input | Canonical |
|------------------|-----------|
| `Asserted`       | `Observed`     |
| `Inferred`       | `Derived`      |
| `Heuristic`      | `Hypothesized` |

A `Hypothesized` fact must carry a `heuristicId`. `confidence` is **per-fact** and
orthogonal to the per-scan `reproLevel` A/B/C grade. See the
[Migration Guide](./MIGRATION-v0.4-to-v0.5.md).

## What's new

- **Dual version fields** — `factAbiVersion` and `modelSchemaVersion` replace the
  single `version`; consumers branch on `factAbiVersion` and ignore unknown fields.
- **HXL gains three node kinds** — `Cond` (ternary / `CASE WHEN`), `Slice`
  (substring / reference modification) and `Index` (subscript), so common logic
  lowers with zero `Opaque` residue. `exprHash` is now `"hxl:" + sha256(canonical
  structural table)[0:16]` — independent of formatting and interning order, stable
  across future node-kind additions.
- **Componentised `behaviouralConfidence`** — `round(100 × min(coverage, rigour ×
  agreement))`, capped by coverage, with the named components round-tripped so any
  validator can recompute the score. No hand-set numbers, no LLM in the path.
- **Richer behavioural assertions** — `id`, `statement`, `evidence[]`, a canonical
  `confidence`, and `predicateExprHash` referencing the spec's `exprTables`; kinds
  narrowed to `invariant | contract | effect | ordering`; `derivedBy ∈ {declared,
  static, trace, learned}` with promotion gates.
- **Conformance toolkit (shipped)** — a published **JSON Schema** (draft 2020-12,
  [`hcs-fact-abi.schema.json`](./hcs-fact-abi.schema.json)) is the authoritative
  structural artifact; a **reference validator** recomputes `factId` / `exprHash` /
  `behaviouralConfidence` / `logicFacts`, checks the promotion gates, coverage cap,
  vocabulary and versions, and returns a precise exit-code contract; a
  **conformance corpus** exercises every exit class, byte-determinism, and
  cross-language `exprHash` equivalence.

## Positioning

The public spec no longer asserts that a system can be regenerated from HCS. HCS
delivers **system discovery & description**, **ecosystem & dependency mapping**,
**behaviour pinning & drift detection**, and a **migration-grade inventory**. Full
behavioural reproduction remains a tracked research direction — not a guarantee.

---

**(c) 2026 Vibgrate. All rights reserved.** | [Changelog](./CHANGELOG.md) | [vibgrate.com](https://vibgrate.com)
