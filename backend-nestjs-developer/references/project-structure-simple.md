# Project Structure — Simple

For small APIs with low traffic and a handful of modules, meant to ship fast. One or two environments (`development`, `production`) are enough — no need for the full three-environment, bounded-context ceremony.

Confirm this tree with the user before creating anything, and create **only** what's listed here.

```
- src/
	- app/
		- app.module.ts
		- app.controller.ts
		- app.service.ts
	- shared/: functions, utilities, and cross-cutting configuration.
		- utils/
		- database/
			- database.module.ts
			- database.provider.ts
	- contexts/: groups the system's bounded contexts. Simple projects can keep
		a single level of modules if separate contexts aren't justified yet.
		- modules/
			- [module]/
				- [module].module.ts
				- [module].controller.ts
				- [module].service.ts
				- [module].entity.ts
				- [module].queries.ts
				- [module].repository.ts
				- dto/
					- req/
						- create-[module]-req.dto.ts
					- res/
						- create-[module]-res.dto.ts
				- tests/: .http files to exercise the module's endpoints.
	- main.ts
- docs/: general project documentation (README, technical decisions).
- .env.development
- .env.production
- .gitignore: must include .env*
```

## Notes

- No `domain/` / `application/` / `infrastructure/` / `presentation/` split here — a module is one flat folder with its module, controller, service, entity, queries, repository, and DTOs side by side. That's the point: less ceremony for a project that doesn't need it yet.
- No `shared/guards/`, `shared/filters/`, `shared/interceptors/`, etc. by default — add them only once the project actually needs auth, custom error formatting, or response wrapping beyond what NestJS gives out of the box. If the project needs the full `main.ts` bootstrap checklist from `references/bootstrap.md`, add the specific `shared/` subfolders that checklist requires (e.g. `shared/filters/ExceptionFormatter.filter.ts`) without adopting the rest of the full structure.
- Configure the `@` alias to `./src` (see `references/alias-tsconfig.md`) from day one, even in a simple project — retrofitting import paths later is pure churn.
- **This structure does not scale by adding more folders on top of it.** When a simple project outgrows it (module count, team size, need for isolated bounded contexts), migrate fully to `references/project-structure-full.md`. The two structures are never mixed within the same project — a project is either simple or full.
