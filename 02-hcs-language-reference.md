![Holistic Code Specification](./assets/hcs-logo-landscape-350.png)

# HCS Language Reference

The HCS DSL uses an **offside (indentation-based) grammar** — identical in spirit to Python. It is designed to be easy to parse, modern-looking, deterministic, and cleanly mappable to a typed AST and canonical YAML.

---

## Lexical Conventions

### Whitespace and Indentation

- Indentation is **semantic** (like Python)
- The indent unit is **2 spaces**. Tabs are **invalid**
- A **Block Header** ends with `:` and must be followed by an indented block on the next non-blank, non-comment line

### Comments

Line comments start with `#` and run to end-of-line.

```hcs
# This is a comment
Define Entity User:  # inline comment
```

### Identifier Shapes

| Shape | Pattern | Used for |
|-------|---------|----------|
| `PascalIdent` | `[A-Z][A-Za-z0-9]*` | Typed nouns (entities, screens, policies) |
| `camelIdent` | `[a-z][A-Za-z0-9]*` | Fields, parameters, JSON keys |
| `SCREAMING_SNAKE` | `[A-Z][A-Z0-9_]*` | Enum values and constants |
| `QualifiedName` | Dot-separated idents | `AuthService.authenticate` |
| `Path` | Filesystem-ish tokens | `src/auth/login.ts` |

The parser accepts a generic `Ident`; casing is enforced by the **validator**, not the grammar.

### Strings

- Double-quoted: `"Hello"`
- Escapes: `\"`, `\\`, `\n`, `\t`

### Numbers and Booleans

- Integers: `0`, `123`
- Decimals: `12.34`
- Booleans: `true`, `false`
- Durations: `500ms`, `30s`, `15m`, `2h`, `7d`

---

## Offside (Indentation) Rule

The token stream includes three virtual tokens:

| Token | Triggered |
|-------|-----------|
| `INDENT` | When indentation increases after a `:` |
| `DEDENT` | When indentation decreases |
| `NEWLINE` | At the end of each logical line |

A line that ends with `:` **must** be followed by an `INDENT` on the next non-empty, non-comment line.

---

## Casing Rules

HCS uses a modern, clear casing system designed to be readable to humans and deterministic for tooling.

> **House style summary:** "HCS uses PascalCase for typed nouns, camelCase for fields, UpperCamelCase for statement keywords, UPPER_SNAKE_CASE for enum values, and kebab-case for files and rule IDs."

### Casing Reference Table

| Item | Format | Example |
|------|--------|---------|
| Full product name | Title Case | Holistic Code Specification |
| Abbreviation | All caps | HCS |
| Version numbers | lowercase v + dot version | HCS v0.1, HCS v1.0 |
| Visible document headings | Title Case | Identity & Access, UI Flows |
| File and folder paths | kebab-case | `hcs/identity-access/`, `spec-modules/` |
| Domains and capabilities | PascalCase | `IdentityAccess`, `OrderManagement` |
| Data entities | PascalCase | `User`, `OrderLine`, `Invoice` |
| Screens | PascalCase | `LoginScreen`, `OrderDetailsScreen` |
| Policies | PascalCase | `AdminAccessPolicy`, `PasswordPolicy` |
| Domain events | PascalCase | `OrderCreated`, `LoginFailed` |
| Integration connectors | PascalCase | `StripeGateway`, `SalesforceConnector` |
| Fields and parameters | camelCase | `userId`, `createdAt`, `passwordHash` |
| Database tables and columns | snake_case | `users`, `order_lines`, `created_at` |
| Statement keywords | UpperCamelCase | `Define`, `When`, `Call`, `FailWith` |
| Enumeration values | UPPER_SNAKE_CASE | `PENDING`, `PAID`, `CANCELLED` |
| Pattern names | PascalCase + Pattern suffix | `CrudTablePattern`, `JwtAuthPattern` |
| Rule and detector IDs | kebab-case or dot.case | `ui.route-detected`, `security.jwt-middleware` |

---

## Core EBNF Grammar

### Top-Level

```ebnf
Document      ::= { (BlankLine | CommentLine | Statement) NEWLINE } ;

BlankLine     ::= (* empty or whitespace only *) ;
CommentLine   ::= "#" { AnyCharExceptNewline } ;

Statement     ::= BlockStatement | SimpleStatement ;
```

### Block Statements

A block statement is a header line ending in `:`, followed by an indented block of statements.

```ebnf
BlockStatement  ::= Header ":" NEWLINE INDENT { Statement NEWLINE } DEDENT ;

Header          ::= Keyword { HeaderAtom } ;

HeaderAtom      ::= WS Atom ;
```

Where `Keyword` is a reserved word and `Atom` is a primitive value.

### Simple Statements

Single-line facts — no trailing `:`.

```ebnf
SimpleStatement ::= Keyword { WS Atom } [ WS Annotations ] ;

Annotations     ::= "@" Annotation { WS "@" Annotation } ;

Annotation      ::= Ident [ "=" Atom ] ;
```

**Annotation examples:**

```hcs
@evidence="src/auth/login.ts:10-42"
@confidence=Observed
@id=HCS-UI-001
```

> **Punctuation rule:** Use `:` only for block headers. Simple statements do **not** have a trailing `:`.

---

## Keywords (Reserved Words)

Keywords are statement verbs. The grammar treats them uniformly; meaning is validated by the statement schema.

**Structural:**
```
Define  Module  Component  Entity  Field  Relationship  Constraint
Screen  Widget  Route  Form  Dialog  Policy  UseCase
```

**Behavioral:**
```
Flow  Step  When  Then  If  Else  ForEach  Try  Catch  Retry
```

**Effects:**
```
Call  Read  Write  Update  Delete  Return  Raise  Handle
Emit  Publish  Subscribe
```

**Security:**
```
Authenticate  Authorize  RequireRole  RequireClaim
ValidateInput  Sanitize  Encrypt  Hash  Audit  FailWith
```

**Patterns:**
```
UsePattern  PatternParam
```

**Metadata:**
```
Doc  Actor  Precondition  Ensures  Property  AppliesTo  Navigate
```

---

## Atoms (Primitive Values)

```ebnf
Atom ::= String | Number | Boolean | Duration | Path | RoutePattern
       | QualifiedName | Ident | Tuple | List | Map ;

Ident         ::= /[A-Za-z_][A-Za-z0-9_]*/ ;
QualifiedName ::= Ident { "." Ident } ;
Path          ::= /[A-Za-z0-9_./-]+/ ;
RoutePattern  ::= "/" { RouteChar } ;
RouteChar     ::= /[A-Za-z0-9_/{}/:-]/ ;

Number        ::= Int | Float ;
Int           ::= /0|[1-9][0-9]*/ ;
Float         ::= Int "." /[0-9]+/ ;
Boolean       ::= "true" | "false" ;
String        ::= "\"" { StringChar } "\"" ;
StringChar    ::= /[^"\\]/ | "\\" ( "\"" | "\\" | "n" | "t" ) ;
Duration      ::= Int ( "ms" | "s" | "m" | "h" | "d" ) ;

Tuple         ::= "(" [ Atom { "," Atom } ] ")" ;
List          ::= "[" [ Atom { "," Atom } ] "]" ;
Map           ::= "{" [ MapEntry { "," MapEntry } ] "}" ;
MapEntry      ::= Ident ":" Atom ;
```

### Type Expressions

Used for field type declarations:

```ebnf
TypeExpr    ::= Ident [ GenericArgs ] [ "?" ] ;
GenericArgs ::= "<" TypeExpr { "," TypeExpr } ">" ;
```

Examples: `UUID`, `String?`, `List<OrderLine>`, `Map<String, Role>`

---

## Statement Schemas

The HCS grammar is intentionally generic. Precision comes from **statement schemas** — per-keyword contracts that declare allowed arguments, allowed children, cardinality rules, and casing constraints.

### AST Node (canonical shape)

Every parsed statement becomes a node in the HCS AST:

```json
{
  "keyword": "Define",
  "args": ["Entity", "User"],
  "annotations": { "id": "HCS-ENT-001", "evidence": "src/user.ts:10-80" },
  "children": [],
  "loc": { "file": "src/entities.hcs", "lineStart": 5, "lineEnd": 12 }
}
```

### Keyword Schema Format

Each keyword is registered in the schema registry with:

| Field | Description |
|-------|-------------|
| `name` | The keyword string |
| `block` | Whether it must have an indented child block |
| `args` | Positional argument types and cardinality |
| `allowedChildren` | Which keywords may appear inside |
| `requiredChildren` | Which keywords must appear inside |
| `constraints` | Extra rules (uniqueness, ordering, co-occurrence) |

### Argument types

```
String, Number, Boolean, Duration, Route, Path,
Ident, PascalIdent, camelIdent, QualifiedName, Enum,
TypeExpr, Any
```

### Constraint kinds

```
UniqueChildByArg    — child keyword args must be unique
MaxChildren         — maximum occurrence of a child keyword
MinChildren         — minimum occurrence of a child keyword
RequireAnnotation   — annotation key must be present
OnlyOneOfChildren   — mutually exclusive children
RequireArgsMatch    — dispatch allowed children by first arg
DisallowChild       — explicitly forbidden child keyword
OrderingPreference  — canonical ordering of child keywords
```

---

## Core Keyword Schemas

### Define (block)

Introduces a typed thing. Its allowed children are dispatched based on the first argument (`kind`).

```
Define <kind: PascalIdent> <name: PascalIdent>:
```

Allowed children depend on `kind`:

| Kind | Allowed Children |
|------|------------------|
| `Entity` | `Field`, `Relationship`, `Constraint`, `Audit`, `UsePattern`, `Doc` |
| `Screen` | `Route`, `Widget`, `Form`, `When`, `UsePattern`, `Doc` |
| `Policy` | `AppliesTo`, `Authorize`, `Audit`, `Doc`, `UsePattern` |
| `Module` | `Component`, `UsePattern`, `Doc` |
| `Component` | `Call`, `Read`, `Write`, `Emit`, `Doc`, `UsePattern` |
| `UseCase` | `Actor`, `Precondition`, `FlowRef`, `Doc` |
| `Flow` | `Step`, `When`, `If`, `Try`, `Doc` |

### Field (simple)

`Field <camelName> : <TypeExpr> [modifiers...]`

```hcs
Field userId : UUID @required=true
Field email : String @required=true @unique=true
```

### Relationship (simple)

```hcs
Relationship Order 1..* OrderLine
```

Format: `Relationship <left: PascalIdent> <cardinality: String> <right: PascalIdent>`

### Route (simple)

```hcs
Route /login
Route /api/orders/{id}
```

### Widget (block)

```hcs
Widget Button SignInButton:
  Property label "Sign In"
  When Click:
    Call AuthService.authenticate
```

### Form (block)

```hcs
Form:
  Field email : Email
  Field password : Secret
  ValidateInput email EmailFormat
```

Allowed children: `Field`, `ValidateInput`, `When`, `Doc`

### When (block)

```hcs
When Click SignInButton:
  Call AuthService.authenticate(email, password)
  If "result == SUCCESS":
    Navigate DashboardScreen
  Else:
    FailWith InvalidCredentials
```

Allowed children: `Then`, `If`, `Else`, `Call`, `Read`, `Write`, `Update`, `Delete`, `Emit`, `FailWith`, `Return`, `Doc`

### If (block) and Else (block)

```hcs
If "balance > 0":
  Call PaymentService.process
Else:
  FailWith InsufficientFunds
```

### Try (block) and Catch (block)

```hcs
Try:
  Call ExternalService.submit
Catch ServiceUnavailable:
  FailWith RetryLater
```

### Call / Read / Write / Update / Delete (simple)

```hcs
Call AuthService.authenticate(email, password)
Read OrderRepository.findById(orderId)
Write OrderRepository.save(order)
Update Order.set(status=PAID)
Delete OrderRepository.remove(orderId)
```

### Emit (simple)

```hcs
Emit OrderCreated order
```

### FailWith (simple)

```hcs
FailWith InvalidCredentials
```

### Authorize (block)

```hcs
Authorize:
  RequireRole ADMIN
  Audit true
```

Allowed children: `RequireRole`, `RequireClaim`, `Audit`, `Doc`

### RequireRole / RequireClaim (simple)

```hcs
RequireRole ADMIN
RequireClaim tenantId "acme-corp"
```

### UsePattern (block)

```hcs
UsePattern CrudTablePattern:
  PatternParam entity Customer
  PatternParam capabilities [LIST, CREATE, EDIT, DELETE]
  PatternParam permissions { viewRole: SALES, editRole: SALES_MANAGER }
```

### PatternParam (simple)

```hcs
PatternParam entity Customer
PatternParam route /customers
```

### Doc (simple)

```hcs
Doc "This entity represents a registered user in the system."
```

---

## Global Validation Rules

These rules apply across all statements, not per keyword.

| Rule | Description |
|------|-------------|
| Evidence discipline | Any statement produced from code **should** have `@evidence`. Every `Define` **must** have evidence. |
| Block correctness | `block: true` keywords must appear with `:`. `block: false` must not have children. |
| Unique field names | `Field` names must be unique inside a `Define Entity`. |
| Max one `Route` per `Screen` | `Route` has max cardinality 1 per `Define Screen`. |
| Max one `AppliesTo` per `Policy` | `AppliesTo` has max cardinality 1 per `Define Policy`. |
| PascalCase for typed nouns | Validated post-parse on `Define Entity/Screen/Policy` names. |
| camelCase for fields | Validated post-parse on `Field` names. |
| UPPER_SNAKE_CASE for enums | Validated post-parse on `Enum` atom values. |

---

## DSL Examples

### Entity

```hcs
Define Entity User: @evidence="src/user.ts:1-120"
  Field userId : UUID @required=true
  Field email : String @required=true @unique=true
  Field passwordHash : String
  Constraint totalLoginAttempts <= 5
  Audit true
```

### Screen and Flow

```hcs
Define Screen LoginScreen: @evidence="src/ui/login.tsx:1-200"
  Route /login
  Form:
    Field email : Email
    Field password : Secret
    ValidateInput email EmailFormat
    ValidateInput password NonEmpty

  When Click SignInButton:
    Call AuthService.authenticate(email, password)
    If "result == SUCCESS":
      Call SessionService.createSession(userId)
      Navigate DashboardScreen
    Else:
      FailWith InvalidCredentials
```

### Policy

```hcs
Define Policy AdminAccessPolicy:
  AppliesTo /admin/*
  Authorize:
    RequireRole ADMIN
    Audit true
```

### Pattern Compression

```hcs
Define Screen CustomerListScreen:
  UsePattern CrudTablePattern:
    PatternParam entity Customer
    PatternParam capabilities [LIST, FILTER, CREATE, EDIT, DELETE]
    PatternParam permissions { viewRole: SALES, editRole: SALES_MANAGER }
```

---

## AST Representation Example

### DSL Input

```hcs
Define Entity User: @evidence="src/user.ts:1-120"
  Field userId : UUID @required=true
  Field email : String @required=true @unique=true
  Audit true
```

### AST Output (JSON)

```json
{
  "keyword": "Define",
  "args": ["Entity", "User"],
  "annotations": { "evidence": "src/user.ts:1-120" },
  "children": [
    {
      "keyword": "Field",
      "args": ["userId", { "type": "TypeExpr", "value": "UUID" }],
      "annotations": { "required": true },
      "children": []
    },
    {
      "keyword": "Field",
      "args": ["email", { "type": "TypeExpr", "value": "String" }],
      "annotations": { "required": true, "unique": true },
      "children": []
    },
    {
      "keyword": "Audit",
      "args": [true],
      "children": []
    }
  ]
}
```

---

**© 2026 Vibgrate. All rights reserved.** | [License](./LICENSE.md) | [vibgrate.com](https://vibgrate.com)
