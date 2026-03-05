![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Pattern System

The pattern system provides reusable, parameterised building blocks that expand into detailed HCS statements. Patterns compress common structures while maintaining full specification fidelity.

---

## Pattern Type System

### Type Universe

```
Type ::= PrimitiveType | ContainerType | SchemaRef | UnionType

PrimitiveType ::= 'string'   -- non-empty UTF-8 text
               |  'int'      -- signed 64-bit integer
               |  'float'    -- IEEE-754 double
               |  'bool'     -- true / false
               |  'date'     -- ISO-8601 date
               |  'datetime' -- ISO-8601 date-time
               |  'uuid'     -- UUID v4
               |  'any'      -- pass-through (no validation)

ContainerType ::= '[' Type ']'                   -- list of Type
               |  '{ string : ' Type '}'         -- map with string keys
               |  Type '?'                       -- optional (may be null)

SchemaRef ::= '@' Identifier                     -- link to a named schema

UnionType ::= Type ('|' Type)+                   -- discriminated union
```

### Built-in Types

| Type | Description | Example |
|------|-------------|---------|
| `string` | Non-empty UTF-8 text | `"UserName"` |
| `int` | 64-bit signed integer | `42`, `-1` |
| `float` | IEEE-754 double | `3.14`, `2.5e10` |
| `bool` | Boolean | `true`, `false` |
| `date` | ISO-8601 date | `"2024-01-15"` |
| `datetime` | ISO-8601 date-time | `"2024-01-15T10:30:00Z"` |
| `uuid` | UUID v4 | `"550e8400-e29b-41d4-a716-446655440000"` |
| `any` | No validation | pass-through |
| `[T]` | List of type T | `["a", "b", "c"]` |
| `{string: T}` | Map with string keys | `{"key": "value"}` |
| `T?` | Optional | `null` or value of T |

---

## Pattern Definition Schema

```yaml
pattern:
  name: PatternName              # PascalCase identifier
  version: "1.0"                 # SemVer
  description: "Brief purpose"
  category: UI | API | Security | Data | Reliability
  
  parameters:
    - name: paramName
      type: string               # from Type Universe
      required: true
      default: "default value"   # if not required
      description: "What this param controls"
  
  expansion:
    - "Statement line with {{paramName}} substitution"
    - "#if condition"
    - "  Conditional content"
    - "#each items as item"
    - "  Repeated content for {{item}}"
```

---

## Template Primitives

### Substitution

Double-brace expressions insert parameter values:

```
Define Service {{serviceName}}
  Implements {{interfaceName}}
```

Given `serviceName: "OrderService"` and `interfaceName: "IOrderService"`:

```
Define Service OrderService
  Implements IOrderService
```

### Conditional: `#if` / `#else` / `#endif`

```
#if requiresAuth
  requires Authentication
#else
  allows Anonymous
#endif
```

Conditions can use:
- Boolean parameters: `#if isAsync`
- Presence checks: `#if description`
- Comparisons: `#if count > 0`
- Negation: `#if !isPublic`

### Iteration: `#each`

```
#each fields as field
  Define Field {{field.name}}
    type {{field.type}}
#endeach
```

Given `fields: [{name: "id", type: "UUID"}, {name: "name", type: "String"}]`:

```
Define Field id
  type UUID
Define Field name
  type String
```

### Local Binding: `#let`

```
#let fullName = serviceName + "Service"
Define Service {{fullName}}
```

### Expression Syntax

Within `{{ }}` expressions:
- String concatenation: `{{a + b}}`
- Property access: `{{entity.name}}`
- Method calls: `{{name.toLowerCase()}}`
- Ternary: `{{isAsync ? "async" : "sync"}}`

---

## Expansion Contracts

Every pattern provides **guarantees** about what its expansion will contain.

### Contract Structure

```yaml
contract:
  emits:
    - kind: Route
      count: 1
      properties:
        httpMethod: POST
        pathTemplate: "/api/{{resource}}"
    - kind: Symbol
      count: "1+"
      properties:
        kind: Method
  
  requires:
    - kind: Entity
      name: "{{entityName}}"
  
  invariants:
    - "All emitted routes have authentication unless allowAnonymous=true"
    - "Field validation matches entity constraints"
```

### Contract Guarantees

| Guarantee | Meaning |
|-----------|---------|
| `emits` | Pattern will produce these HCS constructs |
| `requires` | Pattern expects these to exist (validated) |
| `invariants` | Always-true properties after expansion |

### Count Expressions

- `1` — exactly one
- `"1+"` — one or more
- `"0+"` — zero or more
- `"{{paramName}}"` — count matches parameter

---

## Standard Pattern Library (100 Patterns)

### Group 1 — UI Patterns (ui/, Patterns 1–25)

| # | Pattern | Description |
|---|---------|-------------|
| 1 | `CrudTablePattern` | Full CRUD table: screen, route, load/read, per-capability actions |
| 2 | `DetailViewPattern` | Detail view: screen, route, load/read single entity |
| 3 | `WizardFlowPattern` | Multi-step flow: Define Flow, Step per entry, cancel route |
| 4 | `ModalConfirmPattern` | Confirm/cancel modal: When Click Confirm / Cancel |
| 5 | `InfiniteScrollListPattern` | Infinite scroll: When Load … When Scroll/Append |
| 6 | `FilterSortTablePattern` | Filter + sort table: When Load … When Filter … When Sort |
| 7 | `SearchTypeaheadPattern` | Typeahead search: When Input/Read results with debounce |
| 8 | `DashboardWidgetPattern` | Dashboard widget: Define Screen, When Load/Read data source |
| 9 | `FileUploadPattern` | File upload: When Upload/Call storage service |
| 10 | `InlineEditCellPattern` | Inline cell edit: When Click EditCell, When Blur/Call update |
| 11 | `BulkActionTablePattern` | Bulk-action table: When SelectAll … ForEach/Call action |
| 12 | `TabbedLayoutPattern` | Tabbed layout: Define Screen, Tab per entry |
| 13 | `DrawerPanelPattern` | Side drawer: When Click trigger … Navigate drawer |
| 14 | `KanbanBoardPattern` | Kanban board: Define Screen, When DragDrop/Call update |
| 15 | `TimelineViewPattern` | Timeline: Define Screen, Read events, Render timeline |
| 16 | `MapViewPattern` | Map view: Define Screen, Read markers (lat/lng), Render map |
| 17 | `ChartWidgetPattern` | Chart widget: Define Screen, When Load/Read data source |
| 18 | `FormWithValidationPattern` | Form with validation: fields, When Submit/validate/Call submit |
| 19 | `ToastNotificationPattern` | Toast alerts: When each event → Show toast |
| 20 | `PermissionGuardPattern` | Permission guard: Authorize, RequireRole, Navigate fallback |
| 21 | `EmptyStatePattern` | Empty state: When DataEmpty → Show EmptyState with CTA |
| 22 | `SkeletonLoadingPattern` | Skeleton loader: When Loading → skeleton, When Loaded → Render |
| 23 | `BreadcrumbNavPattern` | Breadcrumb nav: Define Screen, BreadcrumbItem per segment |
| 24 | `SidebarNavPattern` | Sidebar nav: Define Screen, NavItem per entry |
| 25 | `ThemeTogglePattern` | Theme toggle: When Click → Toggle theme, Persist preference |

### Group 2 — API Patterns (api/, Patterns 26–45)

| # | Pattern | Description |
|---|---------|-------------|
| 26 | `CrudApiPattern` | Full CRUD API: route per method, authorize, read/write/delete |
| 27 | `PaginatedListEndpointPattern` | Paginated list: Route GET, Read paginated with filters |
| 28 | `SingleResourceEndpointPattern` | Single-resource GET: Route GET, Read by ID with nested relations |
| 29 | `CommandHandlerPattern` | Command handler: Route POST, Authorize, Call handler, Emit event |
| 30 | `EventSourcedCommandPattern` | Event-sourced command: Load aggregate, Apply, Emit domain event |
| 31 | `ValidationMiddlewarePattern` | Validation middleware: When Request → Validate body, FailWith 422 |
| 32 | `RateLimitPattern` | Rate limiter: Ensure max N requests/window, FailWith 429 |
| 33 | `IdempotencyKeyPattern` | Idempotency: Check key cache, Execute or Return cached response |
| 34 | `HealthCheckPattern` | Health check: Route GET /health, Check services, FailWith 503 |
| 35 | `VersionedApiPattern` | Versioned API: Route prefix /v{{version}}, Deprecation header |
| 36 | `WebhookDeliveryPattern` | Webhook delivery: Read subscribers, ForEach/Call delivery, Retry |
| 37 | `FileDownloadEndpointPattern` | File download: Route GET, Authorize, Stream file (optionally presigned) |
| 38 | `BatchWriteEndpointPattern` | Batch write: Route POST, Validate array, ForEach Write |
| 39 | `MultiTenantApiPattern` | Multi-tenant API: Authorize tenant, Read with tenant scope, FailWith 403 |
| 40 | `GraphQLQueryPattern` | GraphQL query resolver: Query resolver, Read entity |
| 41 | `GraphQLMutationPattern` | GraphQL mutation resolver: Mutation resolver, Validate, Call handler |
| 42 | `gRPCServicePattern` | gRPC service: Define Service, Method per entry, Authorize |
| 43 | `SSEStreamPattern` | SSE stream: Route GET, Open SSE connection, Emit events |
| 44 | `WebSocketChannelPattern` | WebSocket channel: On Connect, Emit per event, On Disconnect |
| 45 | `OpenApiDocPattern` | OpenAPI doc: Generates and serves spec at /openapi.json |

### Group 3 — Security Patterns (security/, Patterns 46–70)

| # | Pattern | Description |
|---|---------|-------------|
| 46 | `JwtAuthPattern` | JWT auth: Authenticate JWT, issuer/audience/algorithms, FailWith 401 |
| 47 | `OAuthCodeFlowPattern` | OAuth 2.0 code flow: Authorize code exchange, Emit TokenIssued |
| 48 | `ApiKeyAuthPattern` | API key auth: Authenticate ApiKey header, FailWith 401 |
| 49 | `MtlsAuthPattern` | mTLS auth: Authenticate mTLS, Validate client cert, FailWith 403 |
| 50 | `SessionCookiePattern` | Session cookie: Authenticate session, Validate, FailWith 401 |
| 51 | `RbacPattern` | RBAC: Authorize, RequireRole, FailWith 403 |
| 52 | `AbacPattern` | ABAC: Authorize, Evaluate attribute policy, FailWith 403 |
| 53 | `OwnershipGuardPattern` | Ownership guard: Read entity, Authorize owner = current user, FailWith 403 |
| 54 | `TenantIsolationPattern` | Tenant isolation: Ensures all queries scoped to current tenant |
| 55 | `CsrfProtectionPattern` | CSRF protection: Validate CSRF token, FailWith 403 |
| 56 | `CorsPattern` | CORS policy: Set CORS policy, Reject disallowed origins |
| 57 | `HstsPattern` | HSTS: Add Strict-Transport-Security header |
| 58 | `ContentSecurityPolicyPattern` | CSP: Add Content-Security-Policy header |
| 59 | `SecretRotationPattern` | Secret rotation: When RotationWindow → Refresh secret, Emit SecretRotated |
| 60 | `AuditLogPattern` | Audit log: When each event → Write audit entry, Emit AuditRecorded |
| 61 | `InputSanitizationPattern` | Input sanitization: When Input → Strip/Encode disallowed characters |
| 62 | `PasswordHashPattern` | Password hash: Call hash(password, algorithm, saltRounds) |
| 63 | `MfaPattern` | MFA: Authenticate second factor, Emit MfaVerified |
| 64 | `BruteForceProtectionPattern` | Brute-force protection: Increment failure counter, FailWith 429 after max |
| 65 | `IpAllowlistPattern` | IP allowlist: Check client IP, FailWith 403 if not in allowlist |
| 66 | `SignedRequestPattern` | Signed request: Validate request signature, FailWith 401 if invalid |
| 67 | `DataMaskingPattern` | Data masking: When Read → Mask fields outside audience |
| 68 | `EncryptAtRestPattern` | Encrypt at rest: Ensures fields encrypted at rest, Decrypt on Read |
| 69 | `TlsTerminationPattern` | TLS termination: Ensures all inbound traffic over TLS ≥ minVersion |
| 70 | `VaultSecretPattern` | Vault secret: Read from Vault, Inject into config, Emit SecretInjected |

### Group 4 — Data Patterns (data/, Patterns 71–85)

| # | Pattern | Description |
|---|---------|-------------|
| 71 | `OutboxPattern` | Transactional outbox: Define outbox table + publisher, ForEach/Emit/Update |
| 72 | `SagaPattern` | Saga: Define Flow, Step, ForEach/Try/Call, Catch/Compensate |
| 73 | `EventSourcingPattern` | Event sourcing: Define event store, Append on command, Rebuild from log |
| 74 | `CqrsPattern` | CQRS: Command side writes to store, Query side reads from read model |
| 75 | `ChangeDataCapturePattern` | CDC: Observe row changes, Publish to downstream |
| 76 | `SoftDeletePattern` | Soft delete: Field deletedAt: Instant?, Ensures queries filter deletedAt |
| 77 | `OptimisticLockPattern` | Optimistic lock: Field version: Int, Check on update, FailWith ConcurrentModification |
| 78 | `PessimisticLockPattern` | Pessimistic lock: Acquire row lock, Release after transaction |
| 79 | `CacheAheadPattern` | Cache-ahead: Read from cache → on miss Read from DB → Populate cache |
| 80 | `WriteThroughCachePattern` | Write-through cache: Write to DB and cache atomically |
| 81 | `MigrationPattern` | Migration: Define Migration, Up/Down steps |
| 82 | `PartitioningPattern` | Partitioning: Ensures table partitioned by partitionKey |
| 83 | `ArchivalPattern` | Archival: Define archiver component, When Schedule → Move old rows |
| 84 | `AuditTrailEntityPattern` | Audit trail entity: Define {{entity}}Audit entity, Append on change |
| 85 | `FullTextSearchPattern` | Full-text search: Ensures full-text index on searchFields, Route GET /search |

### Group 5 — Reliability & Ops Patterns (reliability/, Patterns 86–100)

| # | Pattern | Description |
|---|---------|-------------|
| 86 | `RetryWithBackoffPattern` | Retry with backoff: ForEach attempt, Try/Catch/Sleep, FailWith UpstreamUnavailable |
| 87 | `CircuitBreakerPattern` | Circuit breaker: Track failures, FailWith CircuitOpen, Reset after resetSec |
| 88 | `DeadLetterQueuePattern` | Dead letter queue: ForEach message, Try/Catch → Move to DLQ, Emit DeadLettered |
| 89 | `BulkheadPattern` | Bulkhead: Limits concurrency to maxConcurrent, FailWith ServiceBusy on overflow |
| 90 | `TimeoutPattern` | Timeout: Ensures call completes within timeoutMs, FailWith Timeout |
| 91 | `MetricsPattern` | Metrics: Emit metric on each instrumented event |
| 92 | `TracingPattern` | Tracing: Ensures distributed trace propagation on all outbound calls |
| 93 | `StructuredLoggingPattern` | Structured logging: Ensures all log entries include required structured fields |
| 94 | `AlertingPattern` | Alerting: When Metric passes threshold → Emit Alert → Notify channel |
| 95 | `GracefulShutdownPattern` | Graceful shutdown: On signal, Drain requests, Close DB connections, Exit |
| 96 | `FeatureFlagPattern` | Feature flag: Read flag, When enabled/disabled → Fallback |
| 97 | `ConfigReloadPattern` | Config reload: When Schedule/Reload → Re-read config, Emit ConfigReloaded |
| 98 | `LeaderElectionPattern` | Leader election: Acquire distributed lock, Run as leader, Release on step-down |
| 99 | `SmokeTestPattern` | Smoke test: After Deploy: Call each route, Assert status = expectedStatus |
| 100 | `CanaryDeploymentPattern` | Canary deployment: Route N% to newVersion, Monitor, Rollback if condition met |

---

## Pattern Invocation

Patterns are invoked in HCS with the `Apply` keyword:

```hcs
Apply CrudTablePattern
  entity: Order
  fields:
    - orderId: UUID
    - customerId: UUID
    - total: Decimal
    - status: OrderStatus
  actions:
    - View
    - Edit
    - Delete
  filters:
    - status
    - dateRange
```

### Expansion Process

1. **Parameter Binding** — Map invocation args to pattern parameters
2. **Type Validation** — Check all params match declared types
3. **Template Processing** — Execute `#if`, `#each`, `#let` directives
4. **Substitution** — Replace `{{param}}` expressions
5. **Contract Validation** — Verify emitted constructs match contracts
6. **Flattening** — Merge into parent HCS document

---

## Pattern Composition

Patterns can compose other patterns:

```yaml
pattern:
  name: FullCrudFeature
  parameters:
    - name: entity
      type: "@EntitySchema"
  
  expansion:
    - "Apply CrudTablePattern"
    - "  entity: {{entity.name}}"
    - "  fields: {{entity.fields}}"
    - ""
    - "Apply RestResourcePattern"
    - "  resource: {{entity.name}}"
    - "  operations: [Create, Read, Update, Delete]"
```

### Composition Rules

1. Inner patterns expand before outer patterns
2. Parameter values propagate by reference
3. Nested `#if` blocks maintain proper scope
4. Contract guarantees aggregate (union of `emits`)

---

## Custom Pattern Definition

Projects can define custom patterns in `patterns/` directory:

```
project/
├── patterns/
│   ├── custom/
│   │   └── MyDomainPattern.yaml
│   └── overrides/
│       └── CrudTablePattern.yaml   # local override
└── spec/
    └── domain.hcs
```

### Override Priority

1. Project `patterns/overrides/` (highest)
2. Project `patterns/custom/`
3. Standard library `patterns/` (lowest)

---

## Pattern Versioning

Patterns use semantic versioning:

```yaml
pattern:
  name: RestResourcePattern
  version: "2.1.0"
```

### Version Selection

```hcs
Apply RestResourcePattern@2.1
  ...
```

- `@2` — latest 2.x.x
- `@2.1` — latest 2.1.x  
- `@2.1.0` — exact version

---

## Example: CrudTablePattern

```yaml
pattern:
  name: CrudTablePattern
  version: "1.0.0"
  description: "Data table with CRUD operations"
  category: UI
  
  parameters:
    - name: entity
      type: string
      required: true
    - name: fields
      type: "[{name: string, type: string}]"
      required: true
    - name: actions
      type: "[string]"
      default: ["View", "Edit", "Delete"]
    - name: pagination
      type: bool
      default: true
    - name: pageSize
      type: int
      default: 25
  
  expansion:
    - "Define Screen {{entity}}ListScreen"
    - "  route /{{entity | lowercase}}s"
    - ""
    - "  Define Table {{entity}}Table"
    - "    dataSource {{entity}}Service.list"
    - "#each fields as field"
    - "    Define Column {{field.name}}"
    - "      type {{field.type}}"
    - "#endeach"
    - ""
    - "#if pagination"
    - "    pagination"
    - "      pageSize {{pageSize}}"
    - "#endif"
    - ""
    - "#each actions as action"
    - "    Define Action {{action}}"
    - "      when Click{{action}}"
    - "#if action == 'View'"
    - "      navigate {{entity}}DetailScreen"
    - "#endif"
    - "#if action == 'Edit'"
    - "      navigate {{entity}}EditScreen"
    - "#endif"
    - "#if action == 'Delete'"
    - "      call {{entity}}Service.delete"
    - "      confirm true"
    - "#endif"
    - "#endeach"

  contract:
    emits:
      - kind: Screen
        count: 1
      - kind: Table
        count: 1
      - kind: Column
        count: "{{fields.length}}"
      - kind: Action
        count: "{{actions.length}}"
    requires:
      - kind: Service
        name: "{{entity}}Service"
```

---

## Pattern Validation

Before expansion, patterns are validated:

1. **Schema Validation** — Pattern YAML matches meta-schema
2. **Type Checking** — Parameter types are valid
3. **Template Syntax** — All directives are well-formed
4. **Contract Consistency** — `emits` match expansion output

### Validation Errors

```
ERROR: Pattern CrudTablePattern has invalid expansion:
  Line 15: Unclosed #if block
  Line 23: Unknown parameter 'entityName' (did you mean 'entity'?)
```

---

**© 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
