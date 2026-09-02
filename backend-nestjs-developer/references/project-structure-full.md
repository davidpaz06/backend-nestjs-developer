# Project Structure — Full DDD

For larger, scalable, robust APIs that need detailed error handling and documentation, potentially many modules, and therefore bounded contexts to keep `modules/` from becoming unmanageable. Applies tactical DDD (no CQRS, no Domain Events). Manages three environments: `development`, `staging`, `production`.

Confirm this tree with the user before creating anything, and create **only** what's listed here.

```
- src/
	- app/
		- app.module.ts
		- app.controller.ts
		- app.service.ts
	- shared/
		- decorators/
			- Public.ts
			- CurrentUser.ts
			- Roles.ts
		- filters/
			- ExceptionFormatter.filter.ts
		- guards/
			- AuthGuard.ts
			- RoleGuard.ts
		- interceptors/
			- ResponseFormatter.interceptor.ts
		- strategies/
			- Jwt.strategy.ts
		- pipes/: custom reusable transformation/validation pipes.
		- types/: interfaces and types shared across modules and contexts.
			- jwt-payload.interface.ts
			- role.enum.ts
		- errors/
			- domain-error.base.ts
		- logger/
			- logger.module.ts
			- logger.service.ts
		- scripts/: seeding scripts or any other one-off task.
		- utils/: general-purpose shared functions.
		- database/
			- database.module.ts
			- database.provider.ts
	- contexts/: groups the system's bounded contexts. Can hold any number of
		modules.
		- iam/: authentication context (Identity and Access Management). Only
			contains the `auth` module while the authorization model (RBAC) stays
			static. See references/auth-pattern.md.
			- modules/
				- auth/
					- auth.module.ts
					- tests/
					- docs/
						- swagger/
						- markdown/
					- application/
						- auth.service.ts
						- auth.mapper.ts
					- domain/
						- entities/
							- session.entity.ts
						- value-objects/
							- token.vo.ts
						- errors/
							- invalid-credentials.error.ts
							- session-expired.error.ts
						- auth.repository.interface.ts
					- infrastructure/
						- auth.queries.ts
						- auth.repository.ts
					- presentation/
						- auth.controller.ts
						- dto/
							- req/
								- login.req.dto.ts
								- refresh-token.req.dto.ts
							- res/
								- login.res.dto.ts
		- [context]/
			- modules/
				- [module]/
					- [module].module.ts
					- tests/: .http files to exercise the module's endpoints.
					- docs/: API documentation objects for this module. Every route
						should be documented with DTOs, responses, and related
						protocols. Markdown files documenting a use case or full
						flow (module) can also live here.
						- swagger/
						- markdown/
					- application/
						- [module].service.ts: orchestrates the module's use cases,
						  coordinates repositories, calls domain entities/aggregates.
						  Exposes `public` methods for other contexts to consume and
						  `private` methods for internal use only.
						- [module].mapper.ts: maps incoming DTOs to backend shapes
						  and outgoing data to frontend-facing DTOs.
					- domain/
						- entities/
							- [entity].entity.ts
						- value-objects/
							- [value-object].vo.ts
						- errors/: domain error classes, extend
						  shared/errors/domain-error.base.ts
						- [module].repository.interface.ts
					- infrastructure/
						- [module].queries.ts
						- [module].repository.ts
					- presentation/
						- [module].controller.ts
						- dto/
							- req/
								- create-[module]-req.dto.ts
							- res/
								- create-[module]-res.dto.ts
- main.ts
- .env.development
- .env.staging
- .env.production
- .gitignore: must include .env*
```

## Notes

- `contexts/` can hold as many bounded contexts as the domain needs. `iam/` is the one context this skill treats specially by default (see `references/auth-pattern.md`) — every other context follows the generic `[context]/modules/[module]/` shape above.
- Every module gets all four layers (`domain/`, `application/`, `infrastructure/`, `presentation/`) even if one of them starts out thin — the layering is what keeps cross-context rules (`references/cross-context.md`) enforceable as the project grows.
- A project on this structure never mixes in the simple structure's flatter module shape — if a simple project grows into this, it migrates fully (see `references/project-structure-simple.md`).
- Configure the `@` alias to `./src` (see `references/alias-tsconfig.md`) so nothing here is ever imported with a relative `../../` path.
