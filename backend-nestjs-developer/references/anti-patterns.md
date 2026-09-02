# Anti-Patterns

Actively check for these while implementing and debugging — not only when the user explicitly asks for a review or refactor. If you spot one in existing code while debugging a different issue, name it explicitly even when it isn't the cause of the reported error.

---

## Architecture & DDD

### Anemic Domain Model
Entities that are just getters/setters with no business logic or invariant protection. Example: a public `User.setPassword(p)` that neither hashes nor validates — the entity has no say in what makes it valid.

*Fix:* push the invariant into the entity. `User.setPassword(p)` should hash internally and reject invalid input; the entity is the one place that guarantees it can never exist in an invalid state.

### God Service
A service accumulating responsibilities that don't belong to its module. Example: `UserService` that also sends emails and generates reports.

*Fix:* extract the unrelated responsibility into its own service/module. A service's public surface should map to its module's actual business flow, not "everything that happened to be convenient to put here."

### Repository leaking ORM
`repository.interface.ts` exposes ORM types in its contract. Example: `findAll(): Promise<SelectQueryBuilder<User>>`.

*Fix:* the interface returns domain entities or plain types only. Whatever query-building or ORM-specific type the implementation uses stays inside `infrastructure/`, never crossing into `domain/`.

### Cross-context repository injection
A module injects another context's repository directly instead of going through its public façade. Example: `SaleService` injects `UserRepository` from `iam` directly.

*Fix:* inject the other context's `application/[module].service.ts` and call its public methods. See `references/cross-context.md` for the full rule set.

### Bounded context mal delimitado (Chatty Contexts)
Two contexts calling each other constantly, in both directions, to complete a single operation.

*Fix:* this is a sign the boundary itself is wrong — either merge the two contexts, or extract a third context that owns the shared responsibility. Don't patch it with more calls in both directions.

### Fat Controller
Business logic or direct queries inside the controller instead of delegating to `application/`.

*Fix:* the controller only receives the request, calls the service, and returns what it gets back (through the DTO/interceptor pipeline). Anything resembling a decision or a query belongs in `application/` or `infrastructure/`.

---

## NestJS-specific

### Service Locator via misused `@Global()`
Marking a module `@Global()` when it isn't genuinely transversal, which hides its dependencies from whoever reads the consuming module's imports.

*Fix:* reserve `@Global()` for things that are truly cross-cutting (database, logger) as this skill's defaults already do. A feature module should be imported explicitly wherever it's used.

### Circular dependency between modules, patched with `forwardRef()`
Using `forwardRef()` to paper over a circular dependency instead of redesigning the boundary that created it.

*Fix:* `forwardRef()` is a real, legitimate NestJS tool for genuine circular needs (rare) — the anti-pattern is using it as a first resort. Look at *why* two modules need each other and see if the cross-context rules in `references/cross-context.md` point to a missing third module or a wrong direction of dependency.

### DTO = Entity
Using the same class as the domain/ORM entity and as the request/response DTO, exposing internal fields like `passwordHash`.

*Fix:* DTOs and entities are always separate classes, even when they look nearly identical today. The mapper (`[module].mapper.ts`) is the only place that translates between them.

### Duplicated validation in the service
Manually re-checking format rules in the service that `class-validator` already enforces on the DTO.

*Fix:* validation of shape/format lives on the DTO via decorators, enforced by the global `ValidationPipe`. The service validates business rules (things a decorator can't express), not format.

### Injecting `Request` into the service
Coupling `application/` business logic to the HTTP transport layer by injecting the raw `Request` object into a service.

*Fix:* extract what the service actually needs (a user id, a header value) in the controller or via a decorator (`@CurrentUser`), and pass that plain value into the service. The service should have no idea it's being called over HTTP.

---

## Error Handling & Transactions

### Throw `HttpException` from `application/`
Throwing HTTP exceptions directly from application or domain code instead of a `DomainError`.

*Fix:* application services throw classes extending `DomainError`; only `ExceptionFormatterFilter` translates to an HTTP status. See `references/error-handling.md`.

### Swallowed exception
Catching an error without re-throwing it or logging it — it just disappears.

*Fix:* every catch block either re-throws (possibly wrapped in a more specific error) or logs through `LoggerService` with enough context to diagnose it later. Silence is never acceptable for a caught error.

### Implicit or absent transaction
Multiple related writes with no `QueryRunner` (or the data access library's equivalent), and no rollback path if one write fails.

*Fix:* wrap related writes in a transaction and roll back on failure. If the operation spans contexts, propagate the transaction as an optional parameter through the public methods involved — see `references/cross-context.md`, rule 4.

### Long-running transaction with external I/O inside it
Calling an external service (email, payment gateway) from inside a database transaction block.

*Fix:* keep the transactional block limited to database writes. Do the external call before opening the transaction or after it commits — never while a transaction is open and holding locks.

---

## General Code

### Primitive Obsession
Using raw `string`/`number` for domain concepts instead of a Value Object. Example: `email: string` instead of an `Email` VO that validates its own format.

*Fix:* where a primitive carries domain meaning and validation rules of its own, wrap it in a Value Object under `domain/value-objects/`. Not every string needs one — reserve this for concepts the domain actually cares about getting wrong.

### Magic strings/numbers
Repeated literals with no constant or enum backing them. Example: `'admin'` repeated across the codebase instead of `Role.ADMIN`.

*Fix:* anything repeated more than once, or anything that represents a domain concept (a role, a status, a limit), gets a named constant or enum in `shared/types/`.

### Shotgun surgery
A small business change forces edits across multiple unrelated modules.

*Fix:* usually a sign a concept is duplicated instead of shared, or that a module boundary cuts through a single responsibility. Look for the common piece and extract it to `shared/` or the owning module before the next change makes it worse.

### Copy-paste module
Duplicating a whole module's structure instead of extracting what's actually common to `shared/`.

*Fix:* if two modules are structurally identical except for one piece, extract that piece to `shared/utils/` (or a dedicated shared abstraction) and keep the modules themselves distinct only where they're actually different.
