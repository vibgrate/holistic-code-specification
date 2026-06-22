![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# Migration Guide — HCS v0.4 → v0.5

This guide covers everything a **producer** (scanner/normaliser) or **consumer**
(any tool reading an HCS model) must change to conform to **Fact ABI v0.5 /
Model Schema v0.5**. Most of v0.5 is additive; the one breaking change is the
claim-confidence vocabulary, which ships with an input-alias window so v0.4
models keep loading.

See the full normative requirements in [`../spec/`](../spec/) and the summary in
[`CHANGELOG.md`](./CHANGELOG.md#v050--16-june-2026).

---

## 1. Claim confidence — the one breaking change (FR-CONF-1, FR-MIG-1)

The per-fact `confidence` vocabulary is now exactly:

| Value          | Meaning                                                                 |
|----------------|-------------------------------------------------------------------------|
| `Observed`     | Directly present in a parsed source artifact; evidence contains it verbatim. |
| `Derived`      | Computed deterministically from `Observed` facts by a documented rule.  |
| `Hypothesized` | Produced by a heuristic where evidence is incomplete; lowest trust.     |

The legacy values are **removed from the producer vocabulary**. For the whole
v0.5 line, **consumers MUST accept the deprecated inputs and normalise them**
(the aliases are removed in v0.6):

| Deprecated input | Canonical |
|------------------|-----------|
| `Asserted`       | `Observed`     |
| `Inferred`       | `Derived`      |
| `Heuristic`      | `Hypothesized` |

### Producers
- Stop emitting `Asserted` / `Inferred` / `Heuristic`. Emit only the canonical
  three.
- Any fact with `confidence = Hypothesized` **MUST** carry a non-empty
  `heuristicId` (FR-CONF-2).
- Never define `confidence` in terms of the A/B/C `reproLevel`, or vice versa —
  they are orthogonal (FR-CONF-3).

### Consumers
- Normalise the deprecated aliases on input. The reference CLI does this in its
  fact decompressor (`normalizeConfidenceAliases`): any string-valued
  `confidence` field equal to a deprecated alias is mapped to its canonical
  value before the fact is used. Numeric `confidence` (e.g. COBOL's
  `CobolConfidenceReport.confidence`) is left untouched.

---

## 2. Versioning (FR-VER-1) — additive, but the old field is gone

Replace the single `version` field with the **dual** fields on the model root
and `NormalizerOutput`:

```diff
- "version": "0.4"
+ "factAbiVersion": "0.5",
+ "modelSchemaVersion": "0.5"
```

Consumers **MUST** branch on `factAbiVersion` and ignore unknown fields without
error (FR-VER-3). Both values equal `"0.5"` for this release.

---

## 3. HXL expression IR (FR-HXL, FR-ID-2/3)

### New node kinds (additive)
`Cond`, `Slice`, and `Index` join the closed node set. They are produced where
the source construct exists; existing expressions are unchanged.

| Kind  | Shape                          | Lowered from                                            |
|-------|--------------------------------|---------------------------------------------------------|
| `Cond`  | `{ cond, then, else }`       | ternary `?:`, `CASE WHEN`, value `IF/ELSE`, reducible `EVALUATE` |
| `Slice` | `{ base, start, length? }`   | `X(s:l)`, `SUBSTRING`, constant-foldable `.slice`/`.Substring` |
| `Index` | `{ base, index }`            | array indexing, COBOL `OCCURS` subscript `T(i)`         |

### `exprHash` format change
`exprHash` is now `"hxl:" + sha256(canonicalExpr)[0:16]`, computed over the
**canonical structural** node table (post-order from `root`, re-indexed
operands, typed literals). Two structurally identical expressions — regardless
of source whitespace or interning order — hash identically, and adding node
kinds never perturbs the hash of an expression that does not use them.

> **Action:** if you persisted v0.4 `exprHash` values (bare 16-hex, hashed over
> the raw interned table), recompute them. They are content-addressed, so
> re-running the producer is sufficient.

---

## 4. Behavioural specification (FR-BEH, FR-OBS)

### Assertion kinds
Narrowed to `invariant | contract | effect | ordering`. Map any v0.4 usage:
`golden`/`model` → (re-derive as) `invariant`; `temporal` → `ordering`.

### `derivedBy`
Now `declared | static | trace | learned`. Existing static synthesis stays
`static`. Honour the promotion gates (FR-OBS-2): `learned` ⇒ not `Observed`;
`trace` ⇒ links a `TraceObserved` item and is ≤ `Derived`.

### `behaviouralConfidence` is now computed, not assigned (FR-BEH-2/3)

```
behaviouralConfidence = round( 100 × min( coverage, rigour × agreement ) )
```

- `coverage` = bound surface facts / behaviour-bearing surface facts
- `rigour` = mean over assertions of `w(derivedBy) × strength`
  (`static 1.0, trace 0.8, declared 0.6, learned 0.4`)
- `agreement` = `1 − contradicting pairs / total pairs` (1 when ≤1 assertion)

The named components are round-tripped on the spec (`confidenceComponents`) so a
validator can recompute the score. **Scores will shift** relative to v0.4's
coverage-weighted average — this is expected; the new formula caps the score by
coverage and weights by derivation rigour.

### New/renamed assertion fields
`BehaviouralAssertion` now carries `id` (canonical; `assertionId` kept as a
back-compat alias), `statement`, `evidence[]`, a canonical `confidence`, and —
for invariants — `predicateExprHash` referencing `BehaviouralSpec.exprTables`.
`logicFacts` is now exactly the count of captured logic expressions
(`MethodLogicObserved` + `ValidationRuleObserved` carrying an `Expr`).

---

## 5. ReproLevel (FR-REPRO)

Add `reproLevel ∈ {A,B,C}` to the model root and, where sections are present,
`sectionRepro`. `reproLevel` is the **worst-of** the present section levels, and
each section level is assigned by the Appendix B thresholds, recomputed from
facts. Sections absent from the scan are omitted from the worst-of.

---

## 6. Quick conformance checklist

- [ ] Producers emit only `Observed | Derived | Hypothesized`; `Hypothesized` has `heuristicId`.
- [ ] Consumers normalise `Asserted`/`Inferred`/`Heuristic` on input.
- [ ] `version` replaced by `factAbiVersion` + `modelSchemaVersion` (both `"0.5"`).
- [ ] HXL emits `Cond`/`Slice`/`Index` where applicable; `exprHash` is `hxl:`-prefixed and canonical.
- [ ] `behaviouralConfidence` recomputed via the Appendix G formula; components round-tripped.
- [ ] Assertion `id`/`statement`/`evidence`/`confidence`/`predicateExprHash` present; kinds in the new set.
- [ ] `reproLevel` = worst-of `sectionRepro`.

---

**(c) 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
