![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Changelog

The HCS Fact ABI is **additive within a minor line**: every fact type and field
from an earlier version remains valid and unchanged in later versions, and a
version bump only adds new fact types and new optional fields — never removes or
renames. The single exception in v0.5 is the **confidence-vocabulary change**,
which is explicitly declared breaking below and shipped with an input-alias
window (FR-MIG-1) so v0.4 models keep loading.

| Version | Date | Theme |
|---------|------|-------|
| [v0.5.0](#v050--16-june-2026) | 16 June 2026 | Conformance hardening: confidence vocabulary, dual versions, HXL `Cond`/`Slice`/`Index`, componentised behaviouralConfidence, reproLevel section map |
| [v0.4.0](#v040--15-june-2026) | 15 June 2026 | Logic-as-a-fact: HXL expression IR + logic facts |
| [v0.3.0](#v030--13-june-2026) | 13 June 2026 | Behavioural specification & equivalence layer |
| [v0.2.0](#v020--5-march-2026) | 5 March 2026 | Initial public release: 25 core + 9 COBOL fact types, 100 patterns |

---

## v0.5.0 — 16 June 2026

### Fact ABI v0.5 / Model Schema v0.5 — Conformance hardening

v0.5 makes the model and the `.hcs` view objectively **conformance-testable**:
every derived number is now a deterministic function of facts in the model, the
expression IR closes its remaining gaps, and the trust/version vocabulary is
unified. Targets **Fact ABI v0.5** and **Model Schema v0.5**. See the full
requirements in `spec/` and the migration guide in
[`MIGRATION-v0.4-to-v0.5.md`](./MIGRATION-v0.4-to-v0.5.md).

> **⚠️ Breaking change (FR-CONF-1 / FR-MIG-2): claim-confidence vocabulary.**
> The per-fact `confidence` vocabulary is now exactly `Observed | Derived |
> Hypothesized`. The legacy values `Asserted`, `Inferred`, and `Heuristic` are
> **removed** from the producer vocabulary. For the whole v0.5 line, consumers
> **MUST** accept the deprecated inputs and normalise them (the aliases are
> removed in v0.6):
>
> | Deprecated input (v0.4) | Canonical (v0.5) |
> |-------------------------|------------------|
> | `Asserted`              | `Observed`       |
> | `Inferred`              | `Derived`        |
> | `Heuristic`             | `Hypothesized`   |
>
> A `Hypothesized` fact **MUST** carry a non-empty `heuristicId` (FR-CONF-2).
> `confidence` is **per-fact** and orthogonal to the **per-scan** `reproLevel`
> A/B/C grade — the two are never conflated (FR-CONF-3).

#### Versioning & ABI stability (FR-VER)

- **Dual version fields.** The model root and `NormalizerOutput` now carry both
  `factAbiVersion: "0.5"` (additive fact-ABI line) and `modelSchemaVersion:
  "0.5"` (intermediate-model envelope shape). The legacy single `version` field
  is **removed**.
- **Additive ABI rule.** Within a `factAbiVersion` minor line, producers MUST NOT
  remove, rename, or retype an existing field; only additions are allowed. A
  breaking change increments the major component.
- **Forward-compatibility.** Consumers branch on `factAbiVersion` and ignore
  unknown fields without error.

#### HXL expression IR (FR-HXL / FR-ID)

- **Three new node kinds** close the most common `Opaque` residues:
  - `Cond { cond, then, else }` — ternary `?:`, `CASE WHEN`, value-yielding
    `IF/ELSE`, reducible `EVALUATE`.
  - `Slice { base, start, length? }` — substring / COBOL reference modification
    `X(s:l)`, `SUBSTRING`, constant-foldable `.slice` / `.Substring`.
  - `Index { base, index }` — array/table subscript, COBOL `OCCURS` subscript.

  The reference ternary and the COBOL reference-modification fixtures now lower
  with **zero `Opaque` nodes** (`logicCoverage` 100).
- **`exprHash` is now `"hxl:" + sha256(canonicalExpr)[0:16]`**, computed over the
  **canonical structural** node table (post-order traversal from `root`,
  re-indexed operands, typed literals). It is independent of source formatting
  and of the original interning order, and is **stable across future node-kind
  additions** (FR-ID-3).
- **Honest residue (FR-HXL-4).** Only the minimal non-lowerable sub-tree is
  wrapped as `Opaque { raw }`; producers never guess a non-`Opaque` node.
- **Fidelity & coverage (FR-HXL-5).** `fidelity = 1 − opaqueNodes/totalNodes`
  per expression; `logicCoverage = round(100 × (1 − Σ opaque / Σ total))` over
  all captured logic expressions.

#### Behavioural specification (FR-BEH / FR-OBS)

- **Assertion kinds** narrowed to the normative set `invariant | contract |
  effect | ordering`. (`golden` / `temporal` / `model` are removed.)
- **`derivedBy` taxonomy** is now `declared | static | trace | learned`, with the
  epistemic spine `declared → observed(static, trace) → conjectured(learned)`.
  Promotion gates (FR-OBS-2): a `learned` fact is never `Observed`; a `trace`
  fact links `TraceObserved` evidence and is never above `Derived`.
- **Componentised `behaviouralConfidence`** (FR-BEH-2). The score is now a
  deterministic function of named components — never hand-set:

  ```
  behaviouralConfidence = round( 100 × min( coverage, rigour × agreement ) )
  ```

  where `coverage` is the fraction of behaviour-bearing surface facts bound by
  ≥1 assertion, `rigour = mean(w(derivedBy) × strength)` with
  `w(static)=1.0, w(trace)=0.8, w(declared)=0.6, w(learned)=0.4`, and
  `agreement = 1 − contradicting pairs / total pairs`. The `min(coverage, …)`
  term is the **coverage cap** (FR-BEH-3) — the score can never exceed
  `round(100 × coverage)`. The named components are round-tripped on the spec so
  a validator can recompute it.
- **`BehaviouralAssertion`** now carries `id`, `statement`, `evidence[]`, a
  canonical `confidence`, and (for invariants) `predicateExprHash` referencing an
  entry in `BehaviouralSpec.exprTables`.
- **`logicFacts`** (FR-BEH-4) is exactly the count of captured logic expressions
  (`MethodLogicObserved` + `ValidationRuleObserved` carrying an `Expr`).
- **`TraceObserved`** is added to the schema (additive, populated later): every
  consumer accepts it; producers may omit it in v0.5.

#### Reproducibility levels (FR-REPRO)

- The model root carries `reproLevel ∈ {A, B, C}` and SHOULD carry
  `sectionRepro: Record<{data, integration, security, ui, behaviour}, A|B|C>`.
- `reproLevel` is the **worst-of** the present sections (FR-REPRO-2); each
  section level is assigned by the measurable thresholds in Appendix B and is
  recomputed from facts in the model (FR-REPRO-3).

#### Determinism (FR-DET)

- The **model** is byte-identical for an identical fact set (canonical form:
  UTF-8/LF, sorted keys, deterministic array ordering, canonical numbers). The
  **DSL** is template-deterministic — every token derives from a fact or a fixed
  template. The phrase "minimal nondeterministic wording" is removed.

#### COBOL / mainframe (FR-COB) & DSL (FR-DSL)

- Deterministic `RecordLayoutDeclared` byte layout from PIC/USAGE (e.g. `COMP-3`
  width = `ceil((digits + 1) / 2)`); optional additive `physicalEncoding`.
- `ValidationRuleObserved` is **inapplicable to COBOL** — procedural checks
  surface as `MethodLogicObserved`, so an all-`MethodLogicObserved` `logicFacts`
  count is not under-coverage.
- In-scope cross-references (`FileResourceDeclared.jclDdRef`,
  `CicsTransactionDeclared.program`) resolve to an existing `factId`; out-of-scope
  referents are marked unresolved, never fabricated.
- `RouteChar` grammar fixed to `/[A-Za-z0-9_{}:/-]/`; all nine COBOL fact types
  have a DSL rendering (no model-only fact type).

#### Schema, validator & conformance (FR-SCH / FR-VAL / FR-CNF) — *shipped*

- A published JSON Schema (draft 2020-12) — [`hcs-fact-abi.schema.json`](./hcs-fact-abi.schema.json) —
  is the authoritative structural artifact for the v0.5 behavioural/HXL ABI;
  prose must not contradict it.
- A **reference validator** (`vibgrate hcs validate`, engine in
  `wasm/src/validator.rs`) recomputes `factId` / `exprHash` /
  `behaviouralConfidence` / `logicFacts`, checks the FR-OBS-2 promotion gates,
  the FR-BEH-3 coverage cap, the FR-CONF-1 vocabulary, FR-VER-1 versions, and
  canonical (factId-sorted) ordering, returning the Appendix-I exit codes
  (0 ok · 1 schema · 2 recompute · 3 invariant · 4 non-deterministic).
- A conformance corpus exercises every exit class plus **byte-determinism**
  (FR-CNF-2) and **cross-language `exprHash` equivalence** (FR-CNF-3: `lt` from
  `<`/`LESS THAN`, `eq` from `==`/`EQUAL TO` hash-match).
- `reproLevel` worst-of derivation (FR-REPRO-2) is implemented and tested.

#### Migration & claim scoping (FR-MIG)

- The confidence-vocabulary change is declared breaking with the alias map
  above; see [`MIGRATION-v0.4-to-v0.5.md`](./MIGRATION-v0.4-to-v0.5.md).
- The product claim is scoped to **"migration-grade inventory and
  behaviour-pinning,"** with full behavioural reproduction positioned as a
  roadmap item.

---

## v0.4.0 — 15 June 2026

### Fact ABI v0.4 — Logic as a Fact (additive over v0.3)

The decision logic *inside* methods — historically stored as opaque quoted
strings — is now captured as deterministic, language-agnostic, diffable facts.
**No LLM participates** in any part of this layer.

#### HXL — the HCS Expression Language

A small, closed, typed expression IR encoded as an interned, index-addressed node
table (an SSA-style DAG). Each source language's operators lower to **one canonical
set** (`eq ne lt le gt ge`, `and or`, `add sub mul div mod`, `concat`, `in notin`),
so the same rule from TypeScript, C#, SQL, or COBOL renders identically and is
cross-comparable.

- **Node kinds:** `Lit`, `Ref`, `Member`, `Call`, `Unary`, `Binary`, and `Opaque`
  for the honest residue — anything outside the captured grammar is kept verbatim,
  **never dropped, never guessed**.
- **Round-trips to the DSL:** `If order.total lt 0`; an `Opaque` node renders as a
  visible `«raw»` slice, never silent.
- **`Expr` value kind** added to the language so `If` / `Constraint` /
  `ValidateInput` / `Return` hold a real parsed expression instead of a `String`.

#### New Fact Types (2)

| Fact Type | Description |
|-----------|-------------|
| `MethodLogicObserved` | A guard condition or computed return inside any method body — lowered to HXL and propagated to the surface fact it protects |
| `ValidationRuleObserved` | A declarative validation rule on a schema field (zod / Joi / class-validator / yup), lowered to an HXL invariant linked to the route it validates |

#### Extended Fact Types

- `BehaviouralAssertion` invariants now carry their predicate as an embedded HXL
  node table in `payload.expr`.
- `BehaviouralSpec` now reports `logicFacts` and `logicCoverage` — the count of HXL
  invariants and the aggregate fidelity `1 − (opaque nodes / total nodes)`.

#### Honest, Separate Metrics

`behaviouralConfidence` (surface coverage) and `logicCoverage` (algorithm coverage)
are reported on **separate axes and never blended**. An expression that lowers to a
single `Opaque` node has fidelity `0` and is never promoted to an invariant — no
false confidence from logic we could not faithfully represent.

---

## v0.3.0 — 13 June 2026

### Fact ABI v0.3 — Behavioural Specification Layer (additive over v0.2)

A **behavioural layer** that pins *what the system does* — not just what it
declares — so behavioural change can be described and detected across scans. These
are **derived** facts, produced by the behavioural derivation layer and emitted
through the same emitter, inheriting content-addressed `factId`s.

#### New Fact Types (2)

| Fact Type | Description |
|-----------|-------------|
| `BehaviouralAssertion` | A derived assertion about system behaviour, bound to the structural facts it constrains. `kind ∈ { contract, golden, invariant, temporal, model }` |
| `BehaviouralSpec` | A coverage-honest rollup of all assertions for a scan + application, diffable by `specId` across scans to detect behavioural change |

#### Derivation Discipline

- **Provenance is explicit** via `derivedBy`: `static` (from facts alone, no
  runtime), `trace` (mined from a frozen execution corpus), or `learned` (active
  automata learning). A `trace`/`learned` assertion **must** carry a `corpusRef`;
  one without fails validation (an unbounded claim is forbidden).
- **`behaviouralConfidence` (0–100)** is coverage-weighted over the
  behaviour-bearing surface facts and **capped by coverage**. A green score means
  "a measured slice is pinned," never "complete."
- **No LLM** may participate in deriving an assertion or the confidence number.

---

## v0.2.0 — 5 March 2026

### Fact ABI v0.2

Initial public release of the HCS Fact ABI and supporting specification documents.

#### Core Fact Types (25)

A complete, stable taxonomy of 25 core fact types covering all major concerns of a software system:

| Group | Fact Types |
|-------|-----------|
| **Meta** | `FileIndexed` |
| **Core Code** | `SymbolDeclared`, `CallObserved`, `AnnotationFound`, `ImportObserved` |
| **API** | `RouteDeclared`, `EndpointMetadata`, `ParameterBinding` |
| **Security** | `AuthBoundaryDeclared`, `RoleRequirement`, `ClaimUsage` |
| **Data** | `EntityDeclared`, `FieldDeclared`, `RelationshipDeclared`, `MigrationDeclared` |
| **SQL** | `SqlObjectDeclared`, `SqlUsage`, `SqlStatementObserved`, `DataAccessOperation` |
| **Events** | `EventEmitted`, `EventConsumed` |
| **Integration** | `ExternalServiceCall` |
| **Infrastructure** | `ConfigAccess`, `SecretAccess` |
| **Testing** | `TestCaseDeclared` |

All core types are defined with full TypeScript payload schemas, key field specifications, and factId generation rules.

#### COBOL Extension Fact Types (9)

Nine additional fact types extending the core ABI for mainframe COBOL/JCL systems:

| Fact Type | Description |
|-----------|-------------|
| `CopybookResolved` | COPY statement resolution result with REPLACING pair tracking |
| `RecordLayoutDeclared` | DATA DIVISION record structure with full byte-level PIC clause layout |
| `JclJobDeclared` | JCL job definition: steps, DD statements, symbolic parameters |
| `CicsTransactionDeclared` | CICS transaction-to-program mapping from CSD/RDO definitions |
| `FileResourceDeclared` | COBOL SELECT/ASSIGN + FD file resource declaration |
| `ExecBlockObserved` | Embedded EXEC block (SQL, CICS, DLI, JSON, XML) |
| `CobolProgramDeclared` | Program-level metadata: runtime model, dialect, divisions, call targets |
| `CompileDirectiveObserved` | CBL/PROCESS/SET compile card directives with parsed options |
| `CobolConfidenceReport` | Parse quality metrics: confidence score, parse errors, unresolved copybooks |

#### Pattern System

- **100 canonical patterns** organised in 5 groups:
  - Group 1 — UI (patterns 1–25)
  - Group 2 — API (patterns 26–45)
  - Group 3 — Security (patterns 46–70)
  - Group 4 — Data (patterns 71–85)
  - Group 5 — Reliability & Ops (patterns 86–100)
- Full pattern type system with primitive types, container types, schema references, and union types
- Expansion contracts defining how patterns expand into HCS statements
- Node expander protocol for pattern resolution

#### Specification Documents

| Document | Description |
|----------|-------------|
| `01-hcs-overview.md` | What HCS is, holistic dimensions, evidence graph, reproducibility model |
| `02-hcs-language-reference.md` | DSL grammar, syntax, keywords, casing rules, statement schemas |
| `03-hcs-intermediate-model.md` | Machine-facing canonical model, types, evidence structures |
| `04-hcs-fact-types.md` | Complete fact type reference with payload schemas (25 core + 9 COBOL) |
| `05-hcs-patterns.md` | Pattern type system, expansion contracts, standard library (100 patterns) |

#### Fact ID Generation

```
factId = "hcs:" + factType + ":" + sha256(canonicalKey)[0..15]
```

Deterministic, deduplication-safe identifiers derived from the canonical serialisation of each fact's key fields.

#### Reproducibility Levels

| Level | Meaning |
|-------|---------|
| `A` | Compiler-grade — verified by semantic analysis |
| `B` | Cross-referenced — resolved via name matching |
| `C` | Heuristic — pattern-matched or regex-based |

---

**(c) 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
