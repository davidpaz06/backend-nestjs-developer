# Auth Framework

## authN vs authZ

Authentication (**authN**) and authorization (**authZ**) are different concerns with different lifecycles, and this skill keeps them structurally separate:

- **authN** gets its own context: `contexts/iam/modules/auth/`, structured exactly like any other DDD module — `application/`, `domain/`, `infrastructure/`, `presentation/`. Sessions persist in PostgreSQL by default, through the same data access layer as the rest of the project (`resonance-components/datahandler`). Reach for Redis (`shared/redis/`, mirroring `shared/database/`) only when the project has high-frequency session lookups (high traffic, needs native TTL) — that's an explicit upgrade, not the default.
- **authZ**, under static RBAC (the default — see below), is **not** its own module inside `iam/`. It has no domain: no entity, no repository, no persistent rules — just an enum and a guard reading a JWT claim. Building a DDD layer for that would be empty structure over an enum. It lives in `shared/`.

Building blocks that live in `shared/` because they're transversal to the whole app:

- `shared/guards/`: `AuthGuard`, `RoleGuard`
- `shared/strategies/`: `Jwt.strategy.ts`
- `shared/decorators/`: `@Public`, `@CurrentUser`, `@Roles`
- `shared/types/`: `role.enum.ts`

Guards register globally in `main.ts`. Any route that skips auth needs an explicit `@Public()` — there is no implicit public route.

## Static RBAC (default)

Roles and their permissions are defined in code and embedded in the JWT at login. `RoleGuard` reads the claim — no DB round-trip per request.

```typescript
// shared/types/role.enum.ts
export enum Role {
  ADMIN = 'admin',
  USER = 'user',
}

// shared/guards/RoleGuard.ts
@Injectable()
export class RoleGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const required = this.reflector.get<Role[]>('roles', context.getHandler());
    if (!required) return true;
    const { user } = context.switchToHttp().getRequest();
    return required.some((role) => user.roles?.includes(role));
  }
}
```

## Migrating to dynamic RBAC

Move off static RBAC only when the project genuinely needs one of:

- Permission revocation that takes effect before the token expires.
- Roles/permissions administrable from the database instead of code.

When that's the case, this is an explicit architectural change, not a tweak:

1. Create `contexts/iam/modules/roles/` with real domain: `domain/entities/role.entity.ts`, `domain/entities/permission.entity.ts`, its own repository, the works.
2. `RoleGuard` stops reading the JWT claim directly and instead queries a `RolesPublicService` exposed by the new `roles` module (see `references/cross-context.md` for cross-context communication rules — this is exactly that pattern).

Don't build this preemptively. Static RBAC is the default because most projects don't need live revocation, and adding the dynamic layer early is unjustified structure.
