![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Overview

> **Holistic Code Specification** is a structured-English, evidence-linked, pattern-compressed **description of a software system**, generated deterministically from its source code — a migration-grade inventory of what a system does, what it depends on, and how it behaves.

## Definition

**Holistic Code Specification (HCS)** is a deterministic, human-readable description of a software system, automatically derived from source code (and related assets). It captures the operational reality of a system across functionality, UI interactions, data models, security, integrations, and runtime behaviours — with every claim linked back to the source evidence it came from.

HCS is **not** documentation and **not** summaries. It is a specification artifact.

---

## What HCS Is Made Of

### 1 — HCS Document (human-facing)

A set of **Spec Modules** written in constrained English (the HCS DSL), organised by Domain and Capability — for example: "Identity & Access", "Orders", "Billing", "Admin UI".

Each module is built from **Statements** that look readable but are strictly structured.

### 2 — HCS Intermediate Model (machine-facing)

A canonical JSON/YAML model produced by scanners:

- AST-derived code facts
- Dependency and call graphs
- UI component trees and routes
- DB schema and ORM model
- API contracts and message schemas
- AuthN/AuthZ policies and enforcement points
- Validation rules, invariants, and side effects

The HCS text is a **rendering** of this model. YAML is authoritative; the DSL is rendered from YAML.

### 3 — Evidence Graph (trust & traceability)

Every statement has **Evidence Links**:

| Evidence field | Description |
|----------------|-------------|
| `file` | Repo-relative source path |
| `symbol` | Function/class/method identifiers |
| `lineRange` | Line start and end |
| `configKey` | For settings-driven behaviour |
| `ruleId` | Pattern detector or security rule ID |
| `confidence` | `Observed`, `Derived`, or `Hypothesized` |

---

## What HCS Describes

HCS is built to **discover and describe what a system actually does** — as
deterministic, evidence-linked facts you can drill into. Across a scan it captures:

- Domain behaviour — use-cases and workflows
- Interface contracts — UI, APIs, and message schemas
- Data structures — entities, constraints, relationships
- Security posture — auth flows, authorization rules, boundaries
- Integration & ecosystem — external calls, events, message schemas, data stores, and the dependencies a system relies on
- Non-functional essentials that shape production behaviour — caching, concurrency assumptions

This makes HCS a precise, traceable inventory for understanding, auditing, and
planning the modernization of a system.

> **On reproduction.** HCS makes no claim that a system can be regenerated from
> its specification. Full behavioural reproduction is a research direction tracked
> openly via the reproduction harness — it is *not* a current guarantee.

---

## Canonical Entities in HCS

HCS references a small set of first-class typed nouns:

| Entity | Description |
|--------|-------------|
| `Actor` | A user, system role, or external service |
| `Interface` | Screen, API endpoint, Job, or Message topic |
| `Capability` | A use-case group |
| `Component` / `Module` | A code unit |
| `DataEntity` | A model, table, or document |
| `Operation` | A function, endpoint, or action |
| `Policy` | A security, validation, or compliance rule |
| `Event` | A UI event, domain event, or integration event |

---

## Architecture Overview

```
Codebase
   ↓
Scanner (AST + Rules)
   ↓
HCS Intermediate Model (JSON)
   ↓
Canonical YAML
   ↓
Rendered HCS DSL
```

---

## Holistic Dimensions

HCS must be able to fully describe at least six categories of system behaviour. Together these constitute the "holistic" in Holistic Code Specification.

### A — Functional Behaviour

What the system does and the rules governing it.

| HCS must capture | Examples |
|------------------|----------|
| Use cases and business rules | "User can place an order only if their account is active" |
| Workflows and step sequences | "Order → Payment → Fulfilment → Notification" |
| Edge cases | "If stock is zero, order moves to BACKORDER" |
| Preconditions and postconditions | "Before payment: cart must be non-empty" |
| Error paths and recovery | "On payment failure: retry 3 times, then cancel" |
| Side effects | Writes, events, emails, payment captures |

### B — UI and Interaction Model

How the user interface behaves and how it binds to backend logic.

| HCS must capture | Examples |
|------------------|----------|
| Screens, pages, components | `LoginScreen`, `OrderDetailsScreen` |
| Routes and navigation | `/orders/{id}`, modal open/close |
| Layout structure and composition | Master/detail panes, tab containers, responsive grid breakpoints |
| System menus and command surfaces | Top nav, context menu actions, keyboard-accessible command palette |
| User actions → event handlers → API calls → state changes | Click SignIn → Call AuthService → Navigate Dashboard |
| Form fields, validation, and submission flows | Email format validation before submit |
| Field-level guidance and affordances | Placeholder text, helper copy, tooltips, inline hints, mask formats |
| Permission-driven visibility and enabled state | Edit button disabled unless `SALES_MANAGER` |
| Visual semantics with behavioural meaning | Status colour coding, warning badges, required-field indicators |

### C — Data Model

The structure, constraints, and relationships of all data.

| HCS must capture | Examples |
|------------------|----------|
| Entities, aggregates, fields, types | `Order`, `OrderLine`, `status: OrderStatus` |
| Relationships and cardinality | `Order 1..* OrderLine` |
| Constraints and invariants | `total = sum(lines.lineTotal)` |
| Derived and computed values | `daysOverdue = today - dueDate` |
| Migration history and backward compatibility | EF Core migrations, schema versions |

### D — Security Model

How authentication and authorisation work, and where boundaries are enforced.

| HCS must capture | Examples |
|------------------|----------|
| AuthN: methods, sessions/tokens, lifecycle | JWT, OAuth2, session cookies |
| AuthZ: roles, claims, policies + enforcement points | `[Authorize(Roles="ADMIN")]`, `RequireAuthorization("Policy")` |
| Data protection | Encryption at rest, password hashing algorithms |
| Input validation and sanitisation boundaries | SQL injection defence, XSS sanitisation |
| Audit logging and security-relevant events | Login failure, permission denied |

### E — Integration and Infrastructure

How the system talks to the outside world and how it is configured for production.

| HCS must capture | Examples |
|------------------|----------|
| Endpoints, contracts, versioning | REST, GraphQL, gRPC, OpenAPI spec path |
| Async messaging, events, queues | Kafka topics, Azure Service Bus queues |
| External services and dependencies | Stripe, Salesforce, SendGrid |
| Environment config, feature flags | `appsettings.json` keys, LaunchDarkly flags |
| Rate limits, timeouts, retries | `maxRetries: 3`, `timeoutMs: 5000` |
| Deployment concerns affecting behaviour | Blue/green, sticky sessions |

### F — Quality and Operational Behaviour

Non-functional essentials that change how the system behaves in production and must be known to describe its behaviour fully.

| HCS must capture | Examples |
|------------------|----------|
| Caching behaviours and invalidation | Cache-aside, TTL, invalidate on write |
| Concurrency assumptions | Optimistic locking, row version |
| Idempotency guarantees | Idempotency keys on payment endpoints |
| Observability | Structured logging fields, trace propagation headers |

---

## Coverage Requirement

> An HCS output is incomplete if it is missing any dimension for which relevant facts exist in the codebase.

---

## Behavioural layer — pinning what a system does

The behavioural layer is **not a seventh dimension**. It is a *layer that sits
across* dimensions A (Functional), C (Data), and E (Integration): it takes the
structural facts those dimensions capture and pins *what the system actually does*
with them — response shapes, value invariants, and event/protocol ordering — as
deterministic, evidence-linked assertions that are diffable across scans to
describe and detect behavioural change.

- **Assertions** (`BehaviouralAssertion`) bind a claim about behaviour back to the
  facts it constrains. Each carries a `derivedBy` provenance — `static` (from facts
  alone, no runtime), `trace` (mined from a frozen execution corpus), or `learned`
  (active automata learning) — and a `strength` that rises with derivation rigour.
- **Decision logic** (guards, constraints, computed returns, validation rules) is
  lowered into **HXL**, the HCS Expression Language: a deterministic, typed,
  language-agnostic expression IR. The same rule from any source language collapses
  to one canonical shape, so logic is hashable, diffable, and cross-comparable.
- **Two honest measures, never blended.** `behaviouralConfidence` (0–100) answers
  *"of the behaviour-bearing surface, how much have we pinned with an assertion?"*
  and is **capped by coverage**. `logicCoverage` (0–100) answers *"how much of the
  decision logic did we capture faithfully rather than leave `Opaque`?"* A green
  score means "a measured slice is pinned," never "complete."

No LLM participates in deriving an assertion, an HXL expression, or any confidence
number — the whole point of the layer is predictability.

See [HCS Fact Types · Behavioural & Logic](./04-hcs-fact-types.md#behavioural--logic-fact-types).

---

## Core Principles

These are the **non-negotiables** of HCS.

### 1 — Determinism First

Given the same input repo state and configuration, HCS output must be:

- **Stable** — minimal nondeterministic wording between runs
- **Diff-friendly** — small code changes map to small spec diffs
- **Machine-checkable** — schema validation on every output

### 2 — Traceable Claims Only

Every statement produced by HCS is classified as one of:

| Confidence level | Meaning |
|------------------|---------|
| `Observed` | Directly proven by static facts from the AST or config |
| `Derived` | Proven by rule-based inference over observed facts |
| `Hypothesized` | Allowed only if explicitly labelled and bounded — never silent |

Statements with no evidence link must fail validation.

### 3 — Semantics Over Prose

HCS uses English-like phrasing, but meaning is enforced by a **type system** and a **statement schema**.

- Every keyword has a declared schema (allowed args, allowed children, cardinality rules)
- The renderer produces deterministic phrasing from templates — it never "writes prose"
- Free-form text is only permitted inside `Doc` statements and must be explicitly annotated

### 4 — Pattern Compression Without Losing Fidelity

HCS prefers:

```hcs
UsePattern CrudTablePattern:
  PatternParam entity Customer
  PatternParam capabilities [LIST, CREATE, EDIT, DELETE]
```

over repeating low-level descriptions of listing/sorting/editing/deleting every time.

But patterns must:

- Have **parameters** (typed, validated)
- **Expand** back to canonical meaning deterministically
- Remain **evidence-linked** (pattern instance inherits parent evidence)
- Be **registered** in the pattern library with a typed schema

### 5 — Extensible Vocabulary with Governance

New vocabulary is added via:

1. New **statement types** (keyword + typed slots)
2. New **pattern definitions** (named expansion templates)
3. New **detectors and mappings** (scanner rules)

Never by free-form prose drift.

---

## Semantic Guarantees

HCS reports two **independent, never-conflated** axes (FR-CONF-3):

- **`reproLevel`** — a per-scan / per-section evidence-quality grade `A | B | C`
  (below), assigned by the measurable thresholds in the spec's Appendix B.
- **`confidence`** — a per-fact claim-trust value, exactly one of `Observed |
  Derived | Hypothesized` (FR-CONF-1). A `Hypothesized` fact carries a
  `heuristicId` (FR-CONF-2).

A scan (or section) carries a `reproLevel`; an individual claim carries a
`confidence`. Neither is defined in terms of the other.

### Level A — Compiler-grade

> "A conformant HCS generator reading the same source will produce the same claim."

- Evidence comes exclusively from a deep semantic parser (Roslyn, ts-morph, JDT, or equivalent)
- Symbol names are fully qualified, resolved to their declaration
- Route templates are authoritative (extracted from attributes/decorators)
- Entity fields include physical column names confirmed by ORM metadata or SQL schema
- Auth boundaries are confirmed by middleware registration or attribute analysis

Facts at this level are typically `Observed`.

### Level B — Cross-referenced

> "A conformant generator will produce the same structural claim but some detail fields may vary."

- Evidence comes from tree-sitter (at least one pass) **plus** at least one cross-reference signal
- Symbol names may be simple (not always fully qualified)
- Route templates are extracted from source code but without full semantic resolution
- Entity fields are present but physical column names may be `Derived`
- Auth boundaries are derived from naming conventions or decorator names

Facts at this level are typically `Observed` or `Derived`.

### Level C — Heuristic-only

> "The claim is a best guess based on naming conventions or file structure; it may be wrong."

- Evidence comes from structural heuristics only (file path, class suffix, naming patterns)
- No compiler or AST evidence
- Claims should be reviewed before being actioned

Facts at this level are typically `Hypothesized` (with a `heuristicId`).

---

## Formats

HCS uses a **hybrid format strategy**:

| Format | Role | Who reads it |
|--------|------|--------------|
| YAML (`.yaml`) | Canonical source of truth | Tools, validators, expanders, code generators |
| HCS DSL (`.hcs`) | Human-facing rendered view | Developers, architects, reviewers |
| JSON (`.json`) | Intermediate model and fact ABI | Scanners, normaliser, CI tools |

**YAML is authoritative. The DSL is rendered from YAML.**

---

**© 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
