# Database & Data Access Layer

All database access and query execution goes through **`resonance-components/datahandler`**. Check that library's own documentation for anything not covered here — this section only covers where it plugs into the project structure.

Configuration lives in `src/shared/database/`:

- `database.module.ts` — exposes the configured service. Must carry `@Global()` so it doesn't need re-importing per module.
- `database.provider.ts` — the actual provider configuration (connection, pool, etc.).

```typescript
// shared/database/database.module.ts
@Global()
@Module({
  providers: [databaseProvider],
  exports: [databaseProvider],
})
export class DatabaseModule {}
```

Each module's data access layer is split in two, per the DDD conventions in `references/ddd-conventions.md`:

- `infrastructure/[module].queries.ts` — the raw SQL / query strings.
- `infrastructure/[module].repository.ts` — implements the module's `domain/[module].repository.interface.ts`, using the queries above through `datahandler`.

The repository interface belongs to `domain/` and must never leak ORM or query-builder types — see the "Repository leaking ORM" anti-pattern in `references/anti-patterns.md`. It returns domain entities or primitives, never a query builder or raw row type.
