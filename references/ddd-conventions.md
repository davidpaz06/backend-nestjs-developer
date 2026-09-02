# DDD Conventions

## What a module is

A **module** is a subsystem that works cohesively to carry out one business flow end to end. A module is **not necessarily** an entity or a table — it's organized around a flow, not a schema.

## Modules and Bounded Contexts

`modules/` inside a single folder works for small projects. Once an API has enough modules that a flat `modules/` folder becomes unmanageable, **Bounded Contexts** take over that organizing role: a context is a superset of modules. In a microservices architecture each context would be its own service; in this modular monolith, contexts and modules stay distinct concepts inside one codebase.

## Layering inside a module

Every module in the full DDD structure has four layers:

```
[module]/
  [module].module.ts
  tests/            — .http files exercising the module's endpoints
  docs/
    swagger/        — Swagger decorator objects for this module's routes
    markdown/       — flow/use-case documentation
  application/
    [module].service.ts   — orchestrates use cases; public methods for other
                              contexts, private methods for internal use only
    [module].mapper.ts     — maps DB rows ↔ domain, and domain ↔ response DTOs
                              (snake_case → camelCase happens here)
  domain/
    entities/[entity].entity.ts
    value-objects/[value-object].vo.ts
    errors/                — extend shared/errors/domain-error.base.ts
    [module].repository.interface.ts
  infrastructure/
    [module].queries.ts
    [module].repository.ts
  presentation/
    [module].controller.ts
    dto/
      req/create-[module]-req.dto.ts
      res/create-[module]-res.dto.ts
```

The **entity** owns its invariants — it is not a bag of public getters/setters (see "Anemic Domain Model" in `references/anti-patterns.md`). Validation that belongs to the shape of a single value (an email format, a non-negative amount) belongs in a **Value Object**, not scattered as `string`/`number` checks across services (see "Primitive Obsession").

## Simple projects: single-level modules

For small, low-traffic APIs with a handful of modules, the full context/domain/application/infrastructure/presentation split is often more ceremony than the project needs. In that case, `contexts/modules/[module]/` collapses to a flatter shape — see `references/project-structure-simple.md` for the exact tree. Don't mix the two approaches in the same project: a project either follows the simple structure or the full one.
