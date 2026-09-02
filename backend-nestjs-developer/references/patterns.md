# Patterns Index

Every pattern this skill enforces lives in its own reference file. Load only the file relevant to the task at hand — don't guess at the shape of something covered here.

| Pattern | File | When to load |
|---|---|---|
| Application Bootstrap (`main.ts`) | `references/bootstrap.md` | Scaffolding a project, verifying `main.ts` wiring, response/error shapes |
| Auth Framework — authN vs authZ, static vs dynamic RBAC | `references/auth-pattern.md` | Login/sessions, guards, roles, JWT claims, moving to dynamic RBAC |
| Database & Data Access Layer | `references/database-pattern.md` | Repositories, queries, `datahandler` wiring, `shared/database/` |
| Logging | `references/logging-pattern.md` | Injecting `LoggerService`, wiring `app.useLogger()` |
| DDD Conventions | `references/ddd-conventions.md` | What a module is, bounded contexts, the four layers, simple vs full |
| Cross-Context Communication | `references/cross-context.md` | One context calling another, shared transactions, context boundaries |
| Error Handling — domain vs HTTP | `references/error-handling.md` | Domain errors, `ExceptionFormatterFilter`, transactions |
| Controller Conventions | `references/controller-conventions.md` | Writing or reviewing a controller, method order, guards, throttling |
| `@` Alias / `tsconfig.json` Paths | `references/alias-tsconfig.md` | Project setup, import paths, runtime path resolution |
| Anti-Patterns | `references/anti-patterns.md` | Implementation review, debugging a structural root cause, refactors |
