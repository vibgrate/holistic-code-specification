![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Changelog

The HCS Fact ABI is **additive**: every fact type and field from an earlier
version remains valid and unchanged in later versions. A version bump only adds
new fact types and new optional fields — never removes or renames.

| Version | Date | Theme |
|---------|------|-------|
| [v0.4.0](#v040--15-june-2026) | 15 June 2026 | Logic-as-a-fact: HXL expression IR + logic facts |
| [v0.3.0](#v030--13-june-2026) | 13 June 2026 | Behavioural specification & equivalence layer |
| [v0.2.0](#v020--5-march-2026) | 5 March 2026 | Initial public release: 25 core + 9 COBOL fact types, 100 patterns |

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

A **proof layer** that pins *what the system does* — not just what it declares — so
a migration can be proven behaviour-preserving. These are **derived** facts,
produced by the behavioural derivation layer and emitted through the same emitter,
inheriting content-addressed `factId`s.

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
