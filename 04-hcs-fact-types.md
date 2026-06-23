![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Fact Types (ABI v0.4)

This document defines all fact types in the HCS Fact ABI v0.4 — the JSON interchange format for scanner-produced facts.

There are **25 core fact types**, **9 COBOL extension types**, and **4 behavioural & logic types** (added in v0.3–v0.4).

The Fact ABI is **additive**: every type and field from v0.2 remains valid and unchanged. Later versions only add new types and new optional fields.

---

## Format: NDJSON

Scanners emit one fact per line in **Newline-Delimited JSON** format:

```ndjson
{"factId":"hcs:SymbolDeclared:b3c8a1f244e02a9c","factType":"SymbolDeclared","language":"csharp","scanner":"roslyn","scannerVersion":"0.2.0","emittedAt":"2026-01-15T12:00:00Z","payload":{"qualifiedName":"MyApi.Controllers.UsersController","symbolKind":"class"}}
```

NDJSON was chosen for:
- **Streaming** – each line is independently parseable
- **Append-only logs** – easy incremental scanning
- **Resilience** – one malformed line doesn't break the file

---

## Fact Envelope

Every fact object shares this envelope:

```typescript
type Fact = {
  factId:          string    // "hcs:<factType>:<sha256-prefix-16>"
  factType:        string    // discriminant — one of the types below (core or COBOL extension)
  language?:       string    // source language, e.g. "csharp", "typescript"
  scanner?:        string    // scanner plugin id
  scannerVersion?: string    // semver of scanner
  emittedAt?:      string    // ISO 8601 timestamp
  payload:         object    // fact-specific schema, see individual types
}
```

**Required fields:** `factId`, `factType`, `payload`

---

## Fact Type Index

| # | Fact Type | Category | Key Field(s) | Description |
|---|-----------|----------|--------------|-------------|
| 1 | [FileIndexed](#1--fileindexed) | Meta | `filePath` | One record per scanned source file |
| 2 | [SymbolDeclared](#2--symboldeclared) | Core Code | `qualifiedName` | Named code element (class, method, function, etc.) |
| 3 | [CallObserved](#3--callobserved) | Core Code | `callerSymbol` + `calleeSymbol` | A call from one symbol to another |
| 4 | [AnnotationFound](#4--annotationfound) | Core Code | `targetSymbol` + `annotationName` | Attribute, decorator, or annotation on a symbol |
| 5 | [ImportObserved](#5--importobserved) | Core Code | `sourceFile` + `importTarget` | Import or using-directive in a file |
| 6 | [RouteDeclared](#6--routedeclared) | API | `routeId` | HTTP route / endpoint entry point |
| 7 | [EndpointMetadata](#7--endpointmetadata) | API | `routeId` | Supplemental metadata for a declared route |
| 8 | [ParameterBinding](#8--parameterbinding) | API | `routeId` + `paramName` | A parameter of an HTTP handler |
| 9 | [AuthBoundaryDeclared](#9--authboundarydeclared) | Security | `boundaryId` | Authentication or authorisation boundary |
| 10 | [RoleRequirement](#10--rolerequirement) | Security | `targetSymbol` + `requirement` | Role or policy requirement on a route or symbol |
| 11 | [ClaimUsage](#11--claimusage) | Security | `handlerSymbol` + `claimType` | Usage of a user claim value inside a handler |
| 12 | [EntityDeclared](#12--entitydeclared) | Data | `entityId` | Persistent domain entity (table, document, aggregate) |
| 13 | [FieldDeclared](#13--fielddeclared) | Data | `entityId` + `fieldName` | Field on a declared entity |
| 14 | [RelationshipDeclared](#14--relationshipdeclared) | Data | `fromEntity` + `toEntity` + `cardinality` | Typed relationship between two entities |
| 15 | [MigrationDeclared](#15--migrationdeclared) | Data | `migrationId` | Database schema migration file or class |
| 16 | [SqlObjectDeclared](#16--sqlobjectdeclared) | SQL | `objectName` | SQL asset (table, view, procedure, function, trigger, etc.) |
| 17 | [SqlUsage](#17--sqlusage) | SQL | `callerSymbol` + `sqlObject` | Lightweight reference to a SQL object from code |
| 18 | [SqlStatementObserved](#18--sqlstatementobserved) | SQL | `statementId` | Full structural detail of a SQL statement |
| 19 | [DataAccessOperation](#19--dataaccessoperation) | SQL | `operationId` | Multi-step data-access pipeline in application code |
| 20 | [EventEmitted](#20--eventemitted) | Event | `emitterSymbol` + `eventName` | Domain event emission site |
| 21 | [EventConsumed](#21--eventconsumed) | Event | `handlerSymbol` + `eventName` | Domain event consumption site (subscriber/handler) |
| 22 | [ExternalServiceCall](#22--externalservicecall) | Integration | `callerSymbol` + `serviceRef` | Outbound call to an external service or API |
| 23 | [ConfigAccess](#23--configaccess) | Infrastructure | `callerSymbol` + `configKey` | Access of a configuration key |
| 24 | [SecretAccess](#24--secretaccess) | Infrastructure | `callerSymbol` + `secretRef` | Access of a secrets manager entry |
| 25 | [TestCaseDeclared](#25--testcasedeclared) | Testing | `testSymbol` | Test case or test class declared in the codebase |

**COBOL Extension Types** — emitted by the COBOL/JCL adapter

| # | Fact Type | Category | Key Field(s) | Description |
|---|-----------|----------|--------------|-------------|
| 26 | [CopybookResolved](#26--copybookresolved) | COBOL | `programId` + `copybookName` | COPY statement resolution result |
| 27 | [RecordLayoutDeclared](#27--recordlayoutdeclared) | COBOL | `layoutId` | Record structure with byte-level field layout |
| 28 | [JclJobDeclared](#28--jcljobdeclared) | COBOL | `jobName` | JCL job definition with steps and DD allocations |
| 29 | [CicsTransactionDeclared](#29--cicstransactiondeclared) | COBOL | `transactionId` | CICS transaction-to-program mapping |
| 30 | [FileResourceDeclared](#30--fileresourcedeclared) | COBOL | `resourceId` | COBOL SELECT/FD file resource declaration |
| 31 | [ExecBlockObserved](#31--execblockobserved) | COBOL | `blockId` | Embedded EXEC block (SQL, CICS, DLI, JSON, or XML) |
| 32 | [CobolProgramDeclared](#32--cobolprogramdeclared) | COBOL | `programId` | COBOL program metadata (runtime model, dialect, divisions, etc.) |
| 33 | [CompileDirectiveObserved](#33--compiledirectiveobserved) | COBOL | `sourceFile` + `sourceLine` | CBL/PROCESS/SET compile card directive |
| 34 | [CobolConfidenceReport](#34--cobolconfidencereport) | COBOL | `programId` | Parse quality metrics for a COBOL program |

**Behavioural & Logic Types** — derived by the behavioural layer (v0.3) and the logic/HXL layer (v0.4)

| # | Fact Type | Category | Key Field(s) | Description |
|---|-----------|----------|--------------|-------------|
| 35 | [MethodLogicObserved](#35--methodlogicobserved) | Logic | `logicId` | A guard condition or computed return inside a method body |
| 36 | [ValidationRuleObserved](#36--validationruleobserved) | Logic | `ruleId` | A declarative validation rule on a schema field |
| 37 | [BehaviouralAssertion](#37--behaviouralassertion) | Behaviour | `assertionId` | A derived assertion about what the system *does*, bound to the facts it constrains |
| 38 | [BehaviouralSpec](#38--behaviouralspec) | Behaviour | `specId` | Coverage-honest rollup of all assertions for a scan + application |

---

## 1 — FileIndexed

One record per file discovered by the scanner.

**Required payload fields:** `filePath`, `language`, `sizeBytes`, `sha256`

```typescript
payload: {
  filePath:  string    // (key) workspace-relative path
  language:  string    // e.g. "csharp", "typescript", "sql"
  sizeBytes: number
  sha256:    string    // hex digest
}
```

---

## 2 — SymbolDeclared

A named symbol (class, interface, function, method, property, variable, enum, or type alias) declared in source.

**Required payload fields:** `qualifiedName`, `symbolKind`, `sourceFile`

```typescript
payload: {
  qualifiedName: string    // (key) fully-qualified name
  symbolKind:    "class" | "interface" | "function" | "method" | "property"
               | "variable" | "enum" | "type"
  sourceFile:    string
  sourceLine?:   number
  visibility?:   "public" | "protected" | "internal" | "private" | "unknown"
  isAbstract?:   boolean
  isStatic?:     boolean
  genericArity?: number
}
```

---

## 3 — CallObserved

A call from one symbol to another.

**Required payload fields:** `callerSymbol`, `calleeSymbol`

```typescript
payload: {
  callerSymbol: string    // qualifiedName of caller
  calleeSymbol: string    // qualifiedName of callee
  callKind?:    "direct" | "virtual" | "delegate" | "reflection" | "unknown"
  isAsync?:     boolean
}
```

---

## 4 — AnnotationFound

A structured annotation on a symbol (attribute, decorator, annotation, or comment-tag).

**Required payload fields:** `targetSymbol`, `annotationName`

```typescript
payload: {
  targetSymbol:   string    // qualifiedName of annotated symbol
  annotationName: string    // e.g. "Authorize", "HttpGet", "Table"
  arguments?:     object    // named argument key→value pairs
}
```

---

## 5 — ImportObserved

An import or using-directive observed in a file.

**Required payload fields:** `sourceFile`, `importTarget`

```typescript
payload: {
  sourceFile:   string    // (key) file containing the import
  importTarget: string    // module/namespace being imported
  importKind?:  "module" | "namespace" | "type" | "value" | "wildcard"
  isExternal?:  boolean   // true if from an external package
}
```

---

## 6 — RouteDeclared

An HTTP route / endpoint entry point.

**Required payload fields:** `routeId`, `method`, `template`, `handlerSymbol`

```typescript
payload: {
  routeId:       string    // (key) stable route identifier
  method:        "GET" | "POST" | "PUT" | "PATCH" | "DELETE" | "HEAD" | "OPTIONS" | "ANY"
  template:      string    // URL template, e.g. /users/{id}
  handlerSymbol: string    // qualifiedName of handler method
  sourceFile?:   string
  sourceLine?:   number
}
```

---

## 7 — EndpointMetadata

Supplemental metadata for a declared route (description, tags, deprecation, content types).

**Required payload fields:** `routeId`

```typescript
payload: {
  routeId:     string    // (key) matches RouteDeclared.routeId
  summary?:    string
  tags?:       string[]
  deprecated?: boolean
  produces?:   string[]  // MIME types, e.g. ["application/json"]
  consumes?:   string[]
}
```

---

## 8 — ParameterBinding

A single parameter of an HTTP handler and where it is sourced from.

**Required payload fields:** `routeId`, `paramName`, `source`

```typescript
payload: {
  routeId:   string
  paramName: string
  source:    "route" | "query" | "body" | "header" | "form" | "cookie"
  typeName?: string
  required?: boolean
}
```

---

## 9 — AuthBoundaryDeclared

An authentication or authorisation boundary applied to a scope of routes.

**Required payload fields:** `boundaryId`, `mechanism`

```typescript
payload: {
  boundaryId:      string    // (key)
  mechanism:       "jwt" | "oauth2" | "apikey" | "session" | "mtls" | "none" | "unknown"
  scope?:          "global" | "controller" | "route" | "method"
  handlerSymbol?:  string    // qualifiedName of middleware/filter enforcing it
  allowAnonymous?: boolean
}
```

---

## 10 — RoleRequirement

A role, policy, or scope requirement declared on a route or symbol.

**Required payload fields:** `targetSymbol`, `requirement`

```typescript
payload: {
  targetSymbol:     string    // qualifiedName of the protected symbol
  requirement:      string    // role name, policy name, or scope
  requirementKind?: "role" | "policy" | "claim" | "scope"
}
```

---

## 11 — ClaimUsage

Usage of a user claim value inside a handler (e.g. `User.FindFirst`, `IsInRole`).

**Required payload fields:** `handlerSymbol`, `claimType`

```typescript
payload: {
  handlerSymbol: string    // qualifiedName of the handler reading the claim
  claimType:     string    // e.g. "sub", "email", "role"
  accessKind?:   "read" | "assert" | "unknown"
}
```

---

## 12 — EntityDeclared

A persistent domain entity (DB table, document collection, or aggregate root).

**Required payload fields:** `entityId`, `name`

```typescript
payload: {
  entityId:    string    // (key) stable entity identifier
  name:        string    // class or entity name
  tableName?:  string    // physical table or collection name
  schema?:     string    // DB schema if applicable
  sourceFile?: string
  sourceLine?: number
}
```

---

## 13 — FieldDeclared

A field or column on a declared entity.

**Required payload fields:** `entityId`, `fieldName`, `typeName`

```typescript
payload: {
  entityId:    string
  fieldName:   string
  typeName:    string
  nullable?:   boolean
  isPk?:       boolean    // primary key
  isFk?:       boolean    // foreign key
  columnName?: string     // physical column name if different
  maxLength?:  number
}
```

---

## 14 — RelationshipDeclared

A typed relationship between two entities.

**Required payload fields:** `fromEntity`, `toEntity`, `cardinality`

```typescript
payload: {
  fromEntity:     string
  toEntity:       string
  cardinality:    "one-to-one" | "one-to-many" | "many-to-many" | "many-to-one"
  foreignKey?:    string
  cascadeDelete?: boolean
}
```

---

## 15 — MigrationDeclared

A database schema migration file or class.

**Required payload fields:** `migrationId`

```typescript
payload: {
  migrationId:  string    // (key)
  appliedAt?:   string    // ISO 8601
  reversible?:  boolean
  sourceFile?:  string
}
```

---

## 16 — SqlObjectDeclared

A SQL asset declared or managed by the codebase: table, view, stored procedure, function, trigger, index, sequence, or synonym.

**Required payload fields:** `objectName`, `objectType`

```typescript
payload: {
  objectName:   string    // (key) qualified name, e.g. "dbo.Users"
  objectType:   "table" | "view" | "procedure" | "function" | "trigger"
              | "index" | "sequence" | "type" | "synonym"
  schema?:      string
  sourceFile?:  string
  sourceLine?:  number
  definition?:  string    // raw DDL body text

  // tables and views
  columns?: Array<{
    columnName:    string
    dataType:      string
    nullable?:     boolean
    isPk?:         boolean
    isFk?:         boolean
    defaultValue?: string
    isComputed?:   boolean
    computeExpr?:  string
    isIdentity?:   boolean
    maxLength?:    number
    precision?:    number
    scale?:        number
  }>

  // procedures and functions
  parameters?: Array<{
    paramName:     string
    dataType:      string
    direction?:    "in" | "out" | "inout" | "return"
    defaultValue?: string
    isOptional?:   boolean
  }>

  returnType?:      string    // scalar/table functions
  returnsTable?:    boolean   // table-valued functions

  // triggers
  triggerTiming?:      "before" | "after" | "instead-of"
  triggerEvents?:      Array<"insert" | "update" | "delete">
  triggerTable?:       string
  triggerGranularity?: "row" | "statement"
  triggerCondition?:   string

  // views
  viewQuery?:      string    // links to SqlStatementObserved.statementId
  isMaterialized?: boolean

  referencedObjects?:  string[]
  isSystemGenerated?:  boolean
  checkConstraints?:   string[]

  indexes?: Array<{
    indexName:    string
    columns?:     string[]
    isUnique?:    boolean
    isClustered?: boolean
    filter?:      string
  }>
}
```

---

## 17 — SqlUsage

A lightweight reference fact recording that application code accesses a SQL object. For full query structure use [SqlStatementObserved](#18--sqlstatementobserved).

**Required payload fields:** `callerSymbol`, `sqlObject`, `accessKind`

```typescript
payload: {
  callerSymbol:  string
  sqlObject:     string    // qualified object name
  accessKind:    "select" | "insert" | "update" | "delete" | "exec"
               | "merge" | "truncate" | "unknown"
  usageKind?:    "raw-sql" | "orm" | "orm-linq" | "orm-querybuilder"
               | "dapper" | "ado" | "migration" | "ddl-script" | "unknown"
  statementRef?: string    // links to SqlStatementObserved.statementId
  operationRef?: string    // links to DataAccessOperation.operationId
  sourceFile?:   string
  sourceLine?:   number
}
```

---

## 18 — SqlStatementObserved

A SQL statement or query observed in code, stored procedures, views, or triggers. Captures full structural detail: columns, joins, predicates, subqueries, CTEs, parameter bindings, window functions, and set operations.

**Required payload fields:** `statementId`, `statementKind`, `sourceContext`

```typescript
payload: {
  statementId:      string    // (key)
  statementKind:    "select" | "insert" | "update" | "delete" | "merge"
                  | "exec" | "call" | "create" | "alter" | "drop"
                  | "truncate" | "batch" | "unknown"
  sourceContext:    string    // callerSymbol, procedure name, view name, or trigger name
  sourceFile?:      string
  sourceLine?:      number
  rawSql?:          string    // original SQL text
  detectionMethod?: "raw-sql" | "orm-linq" | "orm-querybuilder" | "dapper"
                  | "ado" | "migration" | "ddl-script" | "sql-file" | "unknown"

  targetObjects?: Array<{
    objectName:  string
    alias?:      string
    objectType?: "table" | "view" | "cte" | "subquery" | "function" | "derived-table" | "unknown"
    schema?:     string
    role?:       "from" | "into" | "update-target" | "delete-target" | "merge-target"
               | "merge-source" | "exec-target" | "unknown"
  }>

  columns?: Array<{
    columnName:     string
    tableRef?:      string
    alias?:         string
    expression?:    string
    isComputed?:    boolean
    aggregateFunc?: string
  }>

  distinct?: boolean
  topN?:     number

  joins?: Array<{
    joinType:     "inner" | "left" | "right" | "full" | "cross" | "natural"
                | "lateral" | "cross-apply" | "outer-apply"
    targetObject: { objectName: string; alias?: string; objectType?: string; schema?: string }
    onCondition?: string
    onColumns?:   Array<{ leftColumn: string; rightColumn: string; operator?: string }>
  }>

  predicates?: SqlPredicate[]
  groupBy?:    string[]
  orderBy?:    Array<{ columnRef: string; direction?: "asc" | "desc" }>
  limit?:      number
  offset?:     number

  ctes?: Array<{
    cteName:      string
    statementId:  string    // links to another SqlStatementObserved
    isRecursive?: boolean
  }>

  parameterBindings?: Array<{
    parameterName:     string    // SQL placeholder, e.g. @userId
    sourceExpression?: string
    direction?:        "in" | "out" | "inout"
    typeName?:         string
    usedInClause?:     string
  }>

  returnsRows?:         boolean
  estimatedComplexity?: "simple" | "moderate" | "complex"
}

// Recursive predicate node used for WHERE, HAVING, ON, CASE WHEN
type SqlPredicate = {
  clause?:       "where" | "having" | "on" | "case-when" | "check" | "filter"
  expression:    string
  operator?:     "=" | "!=" | "<>" | "<" | ">" | "<=" | ">=" | "like" | "ilike"
               | "in" | "not-in" | "between" | "is-null" | "is-not-null"
               | "exists" | "not-exists" | "and" | "or" | "not" | "custom"
  leftOperand?:  string
  rightOperand?: string
  parameterRef?: string    // SQL parameter placeholder
  subqueryRef?:  string    // SqlStatementObserved.statementId
  children?:     SqlPredicate[]
}
```

---

## 19 — DataAccessOperation

A multi-step data-access pipeline in application code (repository method, service method, data-access object). Bridges the gap between thin `SqlUsage` facts and the actual behaviour of ORM-based code.

**Required payload fields:** `operationId`, `callerSymbol`, `semanticKind`

```typescript
payload: {
  operationId:    string    // (key)
  callerSymbol:   string    // qualifiedName of the implementing method
  semanticKind:   "read-one" | "read-list" | "read-paged" | "search"
                | "create" | "update" | "upsert" | "soft-delete" | "hard-delete"
                | "merge" | "bulk-read" | "bulk-write" | "exists-check"
                | "count" | "aggregate" | "custom"
  targetEntity?:  string
  sourceFile?:    string
  sourceLine?:    number
  sourceLineEnd?: number

  parameters?: Array<{
    name:     string
    typeName: string
    purpose?: "identifier" | "filter" | "pagination" | "sort" | "entity-data"
            | "flag" | "cancellation" | "unknown"
  }>

  steps?: DaoStep[]

  returnType?:         string
  returnsCollection?:  boolean
  isAsync?:            boolean
  transactionScope?:   "implicit" | "explicit" | "ambient" | "none"
  relatedStatements?:  string[]    // factIds of SqlStatementObserved facts
}

type DaoStep = {
  stepKind:  "query" | "query-one" | "query-scalar" | "guard" | "mutate"
           | "add" | "remove" | "persist" | "emit-event" | "map"
           | "validate" | "transaction-begin" | "transaction-commit"
           | "transaction-rollback" | "include" | "filter" | "sort"
           | "paginate" | "project" | "aggregate" | "custom"
  description?:         string
  statementRef?:        string      // SqlStatementObserved.statementId
  entityField?:         string
  predicateExpression?: string
  throwsOnFail?:        string
  mutatedFields?:       string[]
  mutationValues?:      Record<string, string>
  sourceExpression?:    string
  sourceLine?:          number
  includeNavigation?:   string
  parameterFlow?: Array<{ fromParam: string; toField: string; via?: string }>
}
```

---

## 20 — EventEmitted

A domain event emission site.

**Required payload fields:** `emitterSymbol`, `eventName`

```typescript
payload: {
  emitterSymbol: string    // qualifiedName of the emitting method
  eventName:     string    // event class name or topic
  eventPayload?: string    // qualifiedName of the event payload type
}
```

---

## 21 — EventConsumed

A domain event consumption site (handler or subscriber).

**Required payload fields:** `handlerSymbol`, `eventName`

```typescript
payload: {
  handlerSymbol: string    // qualifiedName of the handler method
  eventName:     string    // event class name or topic
}
```

---

## 22 — ExternalServiceCall

An outbound call to an external service or API.

**Required payload fields:** `callerSymbol`, `serviceRef`

```typescript
payload: {
  callerSymbol: string    // qualifiedName of caller
  serviceRef:   string    // URL pattern or service identifier
  protocol?:    "http" | "grpc" | "amqp" | "kafka" | "smtp" | "unknown"
  isAsync?:     boolean
}
```

---

## 23 — ConfigAccess

Access of an application configuration key.

**Required payload fields:** `callerSymbol`, `configKey`

```typescript
payload: {
  callerSymbol: string    // qualifiedName of caller
  configKey:    string    // key path, e.g. "ConnectionStrings:Default"
  accessKind?:  "read" | "bind" | "unknown"
}
```

---

## 24 — SecretAccess

Access of a secrets manager entry.

**Required payload fields:** `callerSymbol`, `secretRef`

```typescript
payload: {
  callerSymbol: string    // qualifiedName of caller
  secretRef:    string    // secret name or path
  provider?:    "vault" | "aws-ssm" | "azure-keyvault" | "env" | "file" | "unknown"
}
```

---

## 25 — TestCaseDeclared

A test case or test class declared in the codebase.

**Required payload fields:** `testSymbol`, `testKind`

```typescript
payload: {
  testSymbol:      string    // qualifiedName of test class or method
  testKind:        "unit" | "integration" | "e2e" | "unknown"
  subjectSymbol?:  string    // qualifiedName of the code under test
  framework?:      string    // e.g. "xUnit", "Jest", "pytest"
}
```

---

## COBOL Extension Fact Types

The following 9 fact types extend the core ABI to cover mainframe-specific constructs.

---

## 26 — CopybookResolved

Emitted for every `COPY` statement encountered during preprocessing, recording whether the member was successfully located and inlined.

**Required payload fields:** `programId`, `copybookName`

```typescript
payload: {
  programId:     string    // (key) PROGRAM-ID of the program containing the COPY
  copybookName:  string    // (key) member name as written in source
  sourceFile:    string    // COBOL source file containing the COPY statement
  sourceLine:    number
  resolvedPath:  string | null  // workspace-relative path; null if unresolved
  libraryName?:  string         // OF library clause, if present
  replacing:     { pseudoText: string; by: string }[]
  wasResolved:   boolean
}
```

---

## 27 — RecordLayoutDeclared

Emitted for every level-01 or level-77 group in the DATA DIVISION (WORKING-STORAGE, LOCAL-STORAGE, LINKAGE, or FILE sections) and for copybook-rooted record groups. Contains the full byte-level layout with PIC clause analysis.

**Required payload fields:** `layoutId`

```typescript
payload: {
  layoutId:    string    // (key) "cobol:layout:<program-or-copybook>:<group-name>"
  name:        string
  totalBytes:  number
  encoding:    "ebcdic" | "ascii" | "mixed"
  context:     "working-storage" | "local-storage" | "linkage" | "file" | "copybook"
  sourceFile:  string
  sourceLine:  number
  fields: {
    fieldName:   string
    level:       number
    picture?:    string
    usage?:      string
    byteOffset:  number
    byteLength:  number
    mappedType:  string    // e.g. "string(10)", "zoned-decimal(5,2)", "int32"
    occurs?:     number
    redefines?:  string
    conditions:  { name: string; values: string[] }[]
  }[]
}
```

---

## 28 — JclJobDeclared

Emitted for each JCL job definition. Captures the full job hierarchy: job-level parameters, step sequence, and DD (dataset allocation) statements per step.

**Required payload fields:** `jobName`

```typescript
payload: {
  jobName:        string    // (key)
  sourceFile:     string
  sourceLine:     number
  class?:         string    // JOB CLASS parameter
  msgclass?:      string
  notify?:        string
  symbolicParams: Record<string, string>
  steps: {
    stepName:    string
    programName: string    // EXEC PGM= value
    parm?:       string
    condition?:  string
    ddStatements: {
      ddName:       string
      datasetName?: string
      disposition:  string    // e.g. "OLD", "NEW", "SHR", "(NEW,CATLG,DELETE)"
      accessMode:   "input" | "output" | "input-output" | "append"
    }[]
  }[]
}
```

---

## 29 — CicsTransactionDeclared

Emitted for CICS transaction-to-program mappings sourced from CICS CSD/RDO definitions.

**Required payload fields:** `transactionId`

```typescript
payload: {
  transactionId:      string    // (key) 4-character CICS transaction ID
  programName:        string
  sourceFile?:        string
  conversationModel:  "pseudo-conversational" | "conversational" | "unknown"
  screenMap?:         string    // BMS mapset name, if known
  commareaLayout?:    string    // RecordLayoutDeclared layoutId for the COMMAREA
}
```

---

## 30 — FileResourceDeclared

Emitted for each `SELECT ... ASSIGN` + `FD` entry pair in the ENVIRONMENT and DATA divisions. Captures the file organisation, access mode, and links to the record layout.

**Required payload fields:** `resourceId`

```typescript
payload: {
  resourceId:    string    // (key) "cobol:file:<program>:<fdName>"
  fdName:        string    // FD data-name from FD entry
  organization:  "sequential" | "indexed" | "relative" | "line-sequential" | "unknown"
  accessMode:    "sequential" | "random" | "dynamic"
  assignTarget:  string    // ASSIGN TO operand (device or dataset name)
  recordLayout?: string    // RecordLayoutDeclared layoutId for this file's record
  keyField?:     string    // PRIMARY KEY field for indexed files
  sourceFile:    string
  sourceLine:    number
}
```

---

## 31 — ExecBlockObserved

Emitted for every `EXEC … END-EXEC` block encountered in COBOL source. Covers EXEC SQL, EXEC CICS, EXEC DLI (IMS), and also JSON/XML statement blocks which are piggybacked on the same pipeline.

**Required payload fields:** `blockId`

```typescript
payload: {
  blockId:         string    // (key) "cobol:exec:<program>:<blockKind>:<index>"
  blockKind:       "SQL" | "CICS" | "DLI" | "JSON" | "XML"
  command:         string    // first keyword after EXEC: e.g. "SELECT", "LINK", "GU", "GENERATE"
  rawText?:        string    // verbatim EXEC...END-EXEC text
  containerSymbol: string    // qualifiedName of enclosing program, section, or paragraph
  sourceFile:      string
  sourceLine:      number
}
```

---

## 32 — CobolProgramDeclared

Emitted once per COBOL source file. Aggregates program-level metadata derived from the IDENTIFICATION DIVISION, compile options, and runtime heuristics. Complements the `SymbolDeclared` (symbolKind=`module`) fact that is also emitted for the same PROGRAM-ID.

**Required payload fields:** `programId`

```typescript
payload: {
  programId:      string    // (key) PROGRAM-ID value
  runtimeModel:   "batch" | "cics" | "ims-bmp" | "ims-mpp" | "service" | "unknown"
  dialect:        "ibm-enterprise" | "micro-focus" | "gnucobol" | "unknown"
  sourceFormat:   "fixed" | "free" | "variable"
  encoding:       "ebcdic" | "ascii"
  compileOptions: Record<string, string>    // parsed CBL/PROCESS options dict
  copybooks:      string[]    // resolved copybook paths (workspace-relative)
  calledPrograms: string[]    // static and dynamic CALL targets
  filesAccessed:  string[]    // FD names from SELECT/ASSIGN entries
  dbAccess:       "db2" | "oracle" | "ims-db" | "none" | "unknown"
  isReentrant:    boolean     // IS INITIAL / RENT compile option
  isDynamic:      boolean     // DYNAM compile option
  cobolDivisions: string[]    // divisions present: ["IDENTIFICATION","DATA","PROCEDURE",…]
  sourceFile:     string
}
```

---

## 33 — CompileDirectiveObserved

Emitted for each compile directive card found on the first line(s) of a COBOL source file. CBL and PROCESS cards (IBM Enterprise COBOL) and `$SET` directives (Micro Focus) are captured here rather than as `AnnotationFound` or `ConfigAccess` facts.

**Required payload fields:** `sourceFile`, `sourceLine`

```typescript
payload: {
  sourceFile:    string    // (key)
  sourceLine:    number    // (key) always 1 for CBL/PROCESS; may vary for $SET
  directiveKind: "CBL" | "PROCESS" | "SET"
  rawText:       string    // full text of the directive line
  options:       Record<string, string>    // parsed option name → value pairs
  dialect:       "ibm-enterprise" | "micro-focus" | "unknown"    // inferred from directive kind
  sourceFormat:  "fixed" | "free" | "variable"    // inferred from options if present
}
```

---

## 34 — CobolConfidenceReport

Emitted once per COBOL program after the full extraction pipeline completes. Records parse quality metrics to allow downstream consumers to degrade gracefully when extraction was incomplete.

**Required payload fields:** `programId`

```typescript
payload: {
  programId:           string    // (key) PROGRAM-ID value
  confidenceScore:     number    // 0.0 (failed) – 1.0 (fully parsed)
  parseErrors: {
    rule:    string    // ANTLR rule or extractor phase that failed
    line:    number
    message: string
  }[]
  unresolvedCopybooks: string[]    // member names that could not be located
  partialDivisions:    string[]    // division names that could not be fully parsed
}
```

---

## Behavioural & Logic Fact Types

These types capture **what the system does** — its decision logic and behaviour —
not just what it declares. They are **derived deterministically** (no LLM in the
path): the logic types (`MethodLogicObserved`, `ValidationRuleObserved`) are
emitted by the language scanners as structural evidence, then the behavioural
layer lowers them — and the guard predicates inside `DataAccessOperation` steps —
into assertions.

Decision logic is represented in **HXL**, the HCS Expression Language: a small,
typed, language-agnostic expression IR encoded as an interned, index-addressed
node table. The same rule from TypeScript, C#, SQL, or COBOL lowers to one
canonical shape, so logic is hashable, diffable, and comparable across languages.
Anything outside the captured grammar becomes a single `Opaque` node carrying the
raw source — **never dropped, never guessed**.

### 35 — MethodLogicObserved

A unit of business logic inside any function/method body: a **guard** condition
(`if (cond) throw|return`) or a **computed return** expression. A structural fact;
the behavioural layer lowers `expression` to HXL and links the resulting invariant
to the surface fact it protects.

**Required payload fields:** `logicId`, `containerSymbol`, `kind`, `expression`, `filePath`, `line`

```typescript
payload: {
  logicId:         string    // (key) containerSymbol:kind:line
  containerSymbol: string    // qualifiedName of the enclosing method/function
  kind:            "guard" | "return"
  expression:      string    // the condition (guard) or returned expression — lowered to HXL
  effect?:         "throw" | "return" | null    // for guards
  throwsType?:     string | null                // exception type for throwing guards
  filePath:        string
  line:            number
}
```

### 36 — ValidationRuleObserved

A declarative validation rule on a schema field (zod, Joi, class-validator, yup).
The behavioural layer lowers each rule into an HXL invariant, linked to the route
whose typed request body the schema validates.

**Required payload fields:** `ruleId`, `schemaName`, `fieldName`, `fieldType`, `rule`, `framework`, `filePath`, `line`

```typescript
payload: {
  ruleId:     string    // (key) schemaName:fieldName:rule:arg
  schemaName: string
  fieldName:  string
  fieldType:  "string" | "number" | "boolean" | "array" | "enum" | "object" | "unknown"
  rule:       "min" | "max" | "gt" | "gte" | "lt" | "lte" | "positive"
            | "nonnegative" | "int" | "length" | "nonempty" | "email"
            | "url" | "uuid" | "datetime" | "regex" | "enum"
  arg?:       string | null    // numeric/string argument or regex source
  framework:  "zod" | "joi" | "class-validator" | "yup"
  filePath:   string
  line:       number
}
```

### 37 — BehaviouralAssertion

A derived assertion about what the system **does**, bound to the structural facts
it constrains — the unit of behavioural proof. Coverage-bounded: a `trace` or
`learned` assertion is valid only relative to its frozen corpus (`corpusRef`).
For `invariant` assertions, `expr` carries the predicate as an **HXL node table**.

**Required payload fields:** `id`, `assertionId`, `kind`, `subjectFactIds`, `statement`, `predicate`, `derivedBy`, `confidence`, `strength`, `evidence`

```typescript
payload: {
  id:             string    // (key, FR-BEH-1) content hash over (kind, sorted subjectFactIds, predicate)
  assertionId:    string    // back-compat alias of `id` for pre-v0.5 readers
  kind:           "invariant" | "contract" | "effect" | "ordering"   // FR-BEH-1
  subjectFactIds: string[]  // HCS factIds this assertion constrains (non-empty) — the groundedness link
  statement:      string    // FR-BEH-1 human-facing statement; mirrors the rendered predicate
  predicate:      string    // the constraint, in rendered (round-trip) form
  derivedBy:      "declared" | "static" | "trace" | "learned"        // FR-OBS-1
  confidence:     "Observed" | "Derived" | "Hypothesized"            // FR-CONF-1; gated by derivedBy (FR-OBS-2)
  corpusRef?:     string | null    // required when derivedBy is "trace" or "learned"; null for "static"/"declared"
  strength:       number    // 0..1 — rises with derivation rigour; weights behaviouralConfidence
  evidence:       unknown[] // FR-BEH-1 evidence items (derived facts may carry an empty array)
  predicateExprHash?: string // FR-BEH-1 — references an entry in BehaviouralSpec.exprTables (invariants)
  label?:         string    // presentation only — never folded into the hash
  expr?: {                  // HXL Expression IR (invariants): the logic as a fact
    root:  number
    nodes: Array<{
      k:      "Lit" | "Ref" | "Member" | "Call" | "Unary" | "Binary" | "Cond" | "Slice" | "Index" | "Opaque"
      // operand fields by kind — value/type (Lit), name (Ref/Member), obj (Member),
      // callee/args (Call), op/x (Unary), op/l/r (Binary), cond/then/else (Cond),
      // base/start/length (Slice), base/index (Index), raw (Opaque)
    }>
  }
}
```

### 38 — BehaviouralSpec

The coverage-honest rollup of all `BehaviouralAssertion` facts for a scan +
application. Diffable by `specId` across scans to detect behavioural change.
`behaviouralConfidence` measures **surface** coverage; `logicCoverage` measures
**algorithm** coverage (how much decision logic lowered to faithful, non-`Opaque`
HXL). The two are reported separately, never blended.

**Required payload fields:** `factAbiVersion`, `modelSchemaVersion`, `specId`, `scanId`, `applicationId`, `confidence`, `assertionIds`, `coverage`, `confidenceComponents`, `behaviouralConfidence`, `logicFacts`, `logicCoverage`

```typescript
payload: {
  factAbiVersion:        "0.5"     // FR-VER-1 — additive fact-ABI line
  modelSchemaVersion:    "0.5"     // FR-VER-1 — intermediate-model envelope shape
  specId:                string    // (key) content hash over (scanId, ordered assertionIds)
  scanId:                string
  applicationId:         string
  confidence:            "Observed" | "Derived" | "Hypothesized"   // FR-CONF-1 — claim confidence of the rollup
  assertionIds:          string[]  // factIds of the composing BehaviouralAssertion facts
  coverage: {
    surfaceFacts:  number    // behaviour-bearing surface facts in scope (denominator)
    assertedFacts: number    // surface facts with at least one assertion (numerator)
    byKind?:       Record<string, number>
  }
  confidenceComponents: {    // FR-BEH-2 — named components; a validator recomputes the score
    coverage:  number        // 0..1 — assertedFacts / surfaceFacts
    rigour:    number        // 0..1 — mean(w(derivedBy) × strength)
    agreement: number        // 0..1 — 1 − contradicting pairs / total pairs
  }
  behaviouralConfidence: number    // 0..100 = round(100 × min(coverage, rigour × agreement)); CAPPED BY COVERAGE (FR-BEH-3)
  logicFacts:            number    // FR-BEH-4 — count of captured logic exprs (MethodLogic + ValidationRule)
  logicCoverage:         number    // 0..100 — algorithm coverage = round(100 × (1 − Σ opaque / Σ total))
  exprTables?:           Record<string, { nodes: unknown[]; root: number }>  // HXL tables keyed by exprHash
}
```

---

## FactId Generation

```
factId = "hcs:" + factType + ":" + sha256(canonicalKey)[0..15]
```

The `canonicalKey` is the sorted canonical JSON serialisation of the key field(s) for that fact type (fields marked `(key)` above). This ensures deduplication across scan runs and deterministic merging.

---

## Reproducibility Levels

| Level | Meaning | Evidence Requirements |
|-------|---------|----------------------|
| `A` | **Compiler-grade** — verified by semantic analysis | Full source location, type-checked |
| `B` | **Cross-referenced** — resolved via name matching | Source location, high confidence |
| `C` | **Heuristic** — pattern-matched or regex-based | May lack precise source location |

> `reproLevel` is a **per-scan / per-section** evidence-quality grade. It is
> orthogonal to the **per-fact** `confidence` value (`Observed | Derived |
> Hypothesized`) and is never conflated with it (FR-CONF-3): the `C` level's
> "Heuristic" label describes the evidence method, not a `confidence` value.

---

## NDJSON Examples

```ndjson
{"factId":"hcs:FileIndexed:a1b2c3d4e5f60001","factType":"FileIndexed","language":"csharp","scanner":"roslyn","scannerVersion":"0.2.0","payload":{"filePath":"src/Controllers/UsersController.cs","language":"csharp","sizeBytes":3456,"sha256":"abc123def456"}}
{"factId":"hcs:SymbolDeclared:b3c8a1f244e02a9c","factType":"SymbolDeclared","language":"csharp","scanner":"roslyn","scannerVersion":"0.2.0","payload":{"qualifiedName":"MyApi.Controllers.UsersController","symbolKind":"class","sourceFile":"src/Controllers/UsersController.cs","sourceLine":10,"visibility":"public"}}
{"factId":"hcs:RouteDeclared:f4d9aa11bb220034","factType":"RouteDeclared","language":"csharp","scanner":"roslyn","scannerVersion":"0.2.0","payload":{"routeId":"GET:/api/users/{id}","method":"GET","template":"/api/users/{id}","handlerSymbol":"MyApi.Controllers.UsersController.GetById","sourceFile":"src/Controllers/UsersController.cs","sourceLine":42}}
{"factId":"hcs:AuthBoundaryDeclared:c2d3e4f500000001","factType":"AuthBoundaryDeclared","language":"csharp","scanner":"roslyn","scannerVersion":"0.2.0","payload":{"boundaryId":"boundary:UsersController","mechanism":"jwt","scope":"controller","handlerSymbol":"MyApi.Controllers.UsersController","allowAnonymous":false}}
{"factId":"hcs:EntityDeclared:d4e5f60011000001","factType":"EntityDeclared","language":"csharp","scanner":"roslyn","scannerVersion":"0.2.0","payload":{"entityId":"entity:User","name":"User","tableName":"Users","schema":"dbo","sourceFile":"src/Models/User.cs","sourceLine":5}}
{"factId":"hcs:SqlObjectDeclared:c9f21a3b44e02001","factType":"SqlObjectDeclared","language":"sql","scanner":"sql-parser","scannerVersion":"0.2.0","payload":{"objectName":"dbo.usp_GetOrdersByCustomer","objectType":"procedure","schema":"dbo","sourceFile":"sql/procedures/usp_GetOrdersByCustomer.sql","parameters":[{"paramName":"@CustomerId","dataType":"INT","direction":"in"}],"referencedObjects":["dbo.Orders","dbo.Customers"]}}
{"factId":"hcs:DataAccessOperation:b2b2b2b200000001","factType":"DataAccessOperation","language":"csharp","scanner":"roslyn","scannerVersion":"0.2.0","payload":{"operationId":"dao:ProductRepository.DeleteAsync","callerSymbol":"MyApp.Repositories.ProductRepository.DeleteAsync","semanticKind":"soft-delete","targetEntity":"Product","isAsync":true,"steps":[{"stepKind":"query-one","description":"Find product by ID"},{"stepKind":"guard","description":"Throw if not found","throwsOnFail":"NotFoundException"},{"stepKind":"mutate","mutatedFields":["IsActive"]},{"stepKind":"persist"}]}}
{"factId":"hcs:MethodLogicObserved:f5f5f5f500000001","factType":"MethodLogicObserved","language":"typescript","scanner":"ts-morph","scannerVersion":"0.4.0","payload":{"logicId":"OrderService.deleteAsync:guard:102","containerSymbol":"OrderService.deleteAsync","kind":"guard","expression":"order.total < 0","effect":"throw","throwsType":"InvalidStateException","filePath":"src/order/order.service.ts","line":102}}
{"factId":"hcs:ValidationRuleObserved:a6a6a6a600000001","factType":"ValidationRuleObserved","language":"typescript","scanner":"ts-morph","scannerVersion":"0.4.0","payload":{"ruleId":"CreateUserSchema:email:min:8","schemaName":"CreateUserSchema","fieldName":"email","fieldType":"string","rule":"min","arg":"8","framework":"zod","filePath":"src/user/dto/create-user.schema.ts","line":7}}
{"factId":"hcs:BehaviouralAssertion:e7e7e7e700000001","factType":"BehaviouralAssertion","language":"multi","scanner":"hcs-behavioural","scannerVersion":"0.5.0","payload":{"id":"ba:invariant:9c1f2ab3","assertionId":"ba:invariant:9c1f2ab3","kind":"invariant","subjectFactIds":["hcs:MethodLogicObserved:f5f5f5f500000001"],"statement":"order.total lt 0","predicate":"order.total lt 0","derivedBy":"static","confidence":"Derived","corpusRef":null,"strength":0.8,"evidence":[],"predicateExprHash":"hxl:1b9e0c4f2a7d3c58","label":"guard order.total lt 0","expr":{"root":3,"nodes":[{"k":"Ref","name":"order"},{"k":"Member","obj":0,"name":"total"},{"k":"Lit","value":0,"type":"int"},{"k":"Binary","op":"lt","l":1,"r":2}]}}}
{"factId":"hcs:BehaviouralSpec:e3e3e3e300000001","factType":"BehaviouralSpec","language":"multi","scanner":"hcs-behavioural","scannerVersion":"0.5.0","payload":{"factAbiVersion":"0.5","modelSchemaVersion":"0.5","specId":"spec:scan_8842:v1","scanId":"scan_8842","applicationId":"app_orders","confidence":"Derived","assertionIds":["hcs:BehaviouralAssertion:e7e7e7e700000001"],"coverage":{"surfaceFacts":120,"assertedFacts":78,"byKind":{"contract":64,"invariant":12,"ordering":2}},"confidenceComponents":{"coverage":0.65,"rigour":0.78,"agreement":1},"behaviouralConfidence":65,"logicFacts":12,"logicCoverage":83,"exprTables":{"hxl:1b9e0c4f2a7d3c58":{"root":3,"nodes":[{"k":"Ref","name":"order"},{"k":"Member","obj":0,"name":"total"},{"k":"Lit","value":0,"type":"int"},{"k":"Binary","op":"lt","l":1,"r":2}]}}}}
```

---

**© 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
