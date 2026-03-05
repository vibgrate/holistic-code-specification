![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Changelog

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
