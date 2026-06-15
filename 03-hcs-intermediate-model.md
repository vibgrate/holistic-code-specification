![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Intermediate Model

The HCS Intermediate Model is the **canonical machine-facing representation** of a system, produced by the normaliser from raw facts and consumed by the pattern engine and renderer.

It is language-agnostic, schema-validated, and deterministically sorted.

---

## Top-Level Structure

```typescript
type HcsModel = {
  version: "0.1"
  repo: { root: string; revision?: string }
  symbols:      Record<string, SymbolModel>
  routes:       RouteModel[]
  entities:     Record<string, EntityModel>
  security:     SecurityModel
  integrations: IntegrationsModel
  ui:           UiModel
  graphs:       GraphsModel
  behaviours:   BehavioursModel       // v0.4 — proof layer + HXL expression tables
  evidenceIndex: Record<string, Evidence[]>
  reproLevel:   "A" | "B" | "C"
}
```

---

## Behaviours (v0.4)

The behaviours section carries the **proof layer** — the assertions about what the
system *does* — and the **HXL expression tables** their invariants reference. Logic
is woven into the existing fact graph rather than re-encoded: an assertion's
`subjectFactIds` link back to the structural facts it constrains.

```typescript
type BehavioursModel = {
  assertions: BehaviourAssertionModel[]
  spec:       BehaviourSpecModel
  exprTables: Record<string, HxlExprTable>   // exprHash → interned node table
}

type BehaviourSpecModel = {
  specId:                string
  behaviouralConfidence: number    // 0..100 — surface coverage, capped by coverage
  logicFacts:            number     // count of HXL invariants
  logicCoverage:         number     // 0..100 — algorithm coverage (1 − opaque ratio)
}

type HxlExprTable = {
  nodes: object[]   // flat, index-addressed HXL nodes (SSA-style DAG)
  root:  number
}
```

`behaviouralConfidence` (surface) and `logicCoverage` (algorithm) are **separate
axes** and are never blended. Full field shapes are in
[HCS Fact Types · Behavioural & Logic](./04-hcs-fact-types.md#behavioural--logic-fact-types).

---

## Symbols

```typescript
type SymbolModel = {
  symbolId:   string        // stable hash
  kind:       string        // Class, Method, Function, Property, ...
  name:       string
  fqName:     string
  signature?: string
  access:     "Public" | "Internal" | "Protected" | "Private" | "Unknown"
  isStatic:   boolean
  isAsync:    boolean
  attributes: AttributeModel[]
  confidence: "Observed" | "Derived" | "Heuristic"
  evidence:   Evidence[]
}

type AttributeModel = {
  name:       string
  args?:      any[]
  namedArgs?: Record<string, any>
}
```

**Stable sort key:** `fqName`, then `signature`, then `evidence[0].file`, then `startLine`.

---

## Routes (Endpoints)

```typescript
type RouteModel = {
  routeId:         string    // sha1(httpMethod|pathTemplate|handlerSymbolId)
  httpMethod:      string    // GET / POST / PUT / DELETE / PATCH / ANY
  pathTemplate:    string    // /api/orders/{id}
  handlerSymbolId?: string
  requestType?:    TypeRef
  responseType?:   TypeRef
  bindings:        ParameterBinding[]
  produces?:       string[]
  consumes?:       string[]
  statusCodes?:    number[]
  auth: {
    allowAnonymous: boolean
    requires: AuthZRule[]
  }
  validation:      ValidationRule[]
  evidence:        Evidence[]
}

type ParameterBinding = {
  parameterName: string
  source:        "Route" | "Query" | "Body" | "Header" | "Form" | "Service" | "Unknown"
  typeRef?:      TypeRef
}

type AuthZRule = {
  mode:              "Role" | "Policy" | "Claim" | "AllowAnonymous" | "DenyAnonymous"
  value:             string
  scope:             string
  enforcedBySymbolId?: string
}

type ValidationRule = {
  scope:    string
  field?:   string
  rule:     string
  details?: Record<string, any>
}
```

**Stable sort key:** `pathTemplate` then `httpMethod` then `handlerSymbolId`.

---

## Entities

```typescript
type EntityModel = {
  entityId:  string
  name:      string
  kind:      "EfCore" | "SqlTable" | "SqlView" | "Document" | "Other"
  storage?:  { schema?: string; table?: string }
  fields:    FieldModel[]
  relationships: RelationshipModel[]
  migrations:    MigrationModel[]
  evidence:  Evidence[]
}

type FieldModel = {
  name:        string
  type?:       string
  nullable?:   boolean
  constraints: Record<string, any>  // key+value, stable sort
  evidence:    Evidence[]
  sources:     string[]             // extractor IDs
}

type RelationshipModel = {
  aEntityId:   string
  cardinality: string    // "1..1", "1..*", "*..1", "*..*"
  bEntityId:   string
  fkFields?:   string[]
  cascade?:    string
  evidence:    Evidence[]
}

type MigrationModel = {
  name:     string
  tool?:    "EFCore" | "Flyway" | "Liquibase" | "Custom" | "Unknown"
  evidence: Evidence[]
}
```

**Field sort key:** `name`.  
**Relationship sort key:** `aEntityId|cardinality|bEntityId`.

---

## Security

```typescript
type SecurityModel = {
  authBoundaries: AuthBoundaryModel[]
  authzRules:     AuthZRule[]
  crypto:         CryptoModel[]
  audit:          AuditModel[]
}

type AuthBoundaryModel = {
  boundaryKind:       "Authentication" | "Authorization" | "Cors" | "Csrf" | "Other"
  scope:              string
  enforcedBySymbolId?: string
  evidence:           Evidence[]
}

type CryptoModel = {
  cryptoKind: "Hash" | "Encrypt" | "Decrypt" | "Sign" | "Verify" | "KeyDerive"
  algorithm?: string
  scope:      string
  evidence:   Evidence[]
}

type AuditModel = {
  eventKind: string
  scope:     string
  sink?:     string
  evidence:  Evidence[]
}
```

---

## Integrations

```typescript
type IntegrationsModel = {
  sqlServer: {
    objects:     SqlObjectModel[]
    dependencies: SqlDependencyModel[]
    embeddedSql: SqlEmbeddedModel[]
  }
}

type SqlObjectModel = {
  sqlObjectId: string
  sqlKind:     "Table" | "View" | "Procedure" | "Function" | "Trigger"
  schema?:     string
  name:        string
  evidence:    Evidence[]
}

type SqlDependencyModel = {
  fromSqlObjectId:  string
  toSqlObjectName:  string
  evidence:         Evidence[]
}

type SqlEmbeddedModel = {
  symbolId: string
  rawSql:   string
  parsed:   boolean
  evidence: Evidence[]
}
```

---

## UI

```typescript
type UiModel = {
  screens:    UiScreenModel[]
  events:     UiEventModel[]
  navigation: UiNavigationModel[]
}

type UiScreenModel = {
  screenName: string
  route?:     string
  framework?: string
  evidence:   Evidence[]
}

type UiEventModel = {
  screenName:        string
  eventType:         string
  elementName?:      string
  handlerSymbolId?:  string
  evidence:          Evidence[]
}

type UiNavigationModel = {
  from:              string
  to:                string
  triggerSymbolId?:  string
  evidence:          Evidence[]
}
```

---

## Graphs

```typescript
type GraphsModel = {
  callGraph:       Record<string, string[]>   // callerId → calleeIds[]
  dependencyGraph: Record<string, string[]>   // file → importTargets[]
}
```

Both graphs use stable-sorted value arrays.

---

## Evidence Index

```typescript
evidenceIndex: Record<string, Evidence[]>
// Keys: "Symbol:<id>", "Route:<id>", "Entity:<id>", "SqlObject:<id>", "Policy:<id>"
```

Evidence arrays are deduped by exact tuple `(file, startLine, startCol, endLine, endCol)` and sorted stably.

---

## Evidence Type

The core evidence structure that links HCS claims to source code:

```typescript
type Evidence = {
  file:       string    // repo-relative path
  startLine:  number    // 1-based
  startCol:   number    // 0-based
  endLine:    number    // 1-based
  endCol:     number    // 0-based
}
```

---

## TypeRef

Used wherever a type reference is needed in routes, fields, or calls.

```typescript
type TypeRef = {
  kind:     "Type" | "Symbol" | "Entity"
  id:       string    // stable hash
  name:     string
  fqName?:  string
}
```

---

## ReproLevel

Every top-level model object carries a `reproLevel` field indicating evidence quality:

```typescript
type ReproLevel = 'A' | 'B' | 'C';
```

| Level | Criteria | Meaning |
|-------|----------|---------|
| `A` | ≥95% routes resolved, ≥80% calls resolved, ≥90% entities have fields, ≥95% evidence ranges | Reproducible — suitable for migration and reimplementation |
| `B` | ≥70% routes resolved, ≥70% entities have fields | Migratable — good for inventory, partial HCS |
| `C` | Below Level B thresholds | Document-only — mostly indexing |

---

## YAML Format Examples

### Entity

```yaml
entity:
  name: User
  fields:
    - name: userId
      type: UUID
      required: true
    - name: email
      type: String
      required: true
      unique: true
    - name: passwordHash
      type: String
  audit: true
```

### Screen

```yaml
screen:
  name: LoginScreen
  route: /login
  form:
    fields:
      - email
      - password
  actions:
    - when: Submit
      call: AuthService.authenticate
      onSuccess: Navigate DashboardScreen
      onFailure: Show InvalidCredentials
```

### Policy

```yaml
policy:
  name: AdminAccessPolicy
  appliesTo: /admin/*
  requireRole: ADMIN
```

---

## Output File Layout

```
hcs/
├── model/
│   ├── hcs.model.json         # canonical intermediate model
│   └── hcs.model.yaml         # canonical intermediate model (YAML view)
├── spec/
│   ├── identity-access.hcs    # rendered DSL per domain
│   ├── orders.hcs
│   ├── billing.hcs
│   └── ...
├── patterns/
│   ├── ui/
│   │   ├── CrudTablePattern.yaml
│   │   └── ...
│   ├── security/
│   └── data/
└── expanded/
    └── hcs.expanded.json      # debug: pattern-expanded AST
```

---

## Normalizer Output Format

```typescript
interface NormalizerOutput {
  version:   '0.1';
  scanId:    string;             // sha256 of all factIds, sorted
  scanDate:  string;             // ISO 8601
  model:     IntermediateModel;
  warnings:  NormalizerWarning[];
  stats: {
    factsIngested:     number;
    symbolsProduced:   number;
    routesProduced:    number;
    entitiesProduced:  number;
    warningCount:      number;
  };
}

interface NormalizerWarning {
  code:     string;             // e.g. 'UNKNOWN_ROUTE_REF'
  message:  string;
  factId?:  string;
}
```

---

## Determinism Guarantee

The Intermediate Model is fully deterministic:

1. Input facts are sorted by `factId` before processing
2. All `Set` / `Map` structures are iterated in insertion order after the sort
3. All upsert operations use stable comparison — alphabetical on the key field
4. ReproLevel computation depends only on scanner name strings (stable)

Given the same set of input facts (regardless of order), the model is always byte-identical.

---

## Merge Priority

When the same logical entity can be observed by multiple scanners, the **Merge Priority** rule applies:

```
Roslyn > ts-morph > tree-sitter > heuristic
```

Higher-priority facts overwrite lower-priority facts for the same key field. Supplemental (non-key, non-conflicting) fields from lower-priority sources are merged in additively.

---

**© 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
