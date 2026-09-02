# Naming Rules

File and class naming conventions across every layer. Consistency here is what makes a project navigable without reading every file — a name should tell you the layer and the responsibility before you open it.

---

## General rule

**Files:** `kebab-case`, suffixed with the role: `[name].[role].ts`.
**Classes:** `PascalCase`, same suffix pattern, without the file's hyphenation: `[Name][Role]`.

| File | Class |
|---|---|
| `user.module.ts` | `UserModule` |
| `user.controller.ts` | `UserController` |
| `user.service.ts` | `UserService` |
| `user.entity.ts` | `User` (entity classes are named after the concept, not suffixed `Entity`) |
| `user.repository.interface.ts` | `UserRepositoryInterface` |
| `user.repository.ts` | `UserRepository` |
| `user.queries.ts` | (exported query constants/functions, no class) |
| `user.mapper.ts` | `UserMapper` |

---

## Domain layer

| File | Class | Notes |
|---|---|---|
| `domain/entities/[entity].entity.ts` | `[Entity]` | e.g. `session.entity.ts` → `Session` |
| `domain/value-objects/[value-object].vo.ts` | `[ValueObject]` | e.g. `email.vo.ts` → `Email` |
| `domain/errors/[error-name].error.ts` | `[ErrorName]Error` | e.g. `insufficient-stock.error.ts` → `InsufficientStockError`. Always extends `DomainError` |
| `domain/[module].repository.interface.ts` | `[Module]RepositoryInterface` | Contract only, no ORM types leaked |

## Application layer

| File | Class | Notes |
|---|---|---|
| `application/[module].service.ts` | `[Module]Service` | Public methods = cross-context/external API; private methods = internal use cases |
| `application/[module].mapper.ts` | `[Module]Mapper` | DB row ↔ domain, domain ↔ response DTO. snake_case → camelCase happens here |

## Infrastructure layer

| File | Class | Notes |
|---|---|---|
| `infrastructure/[module].queries.ts` | — | Raw query strings/builders, consumed only by the repository |
| `infrastructure/[module].repository.ts` | `[Module]Repository` | Implements `domain/[module].repository.interface.ts` |

## Presentation layer

| File | Class | Notes |
|---|---|---|
| `presentation/[module].controller.ts` | `[Module]Controller` | Thin — delegates to `application/` |
| `presentation/dto/req/[action]-[module]-req.dto.ts` | `[Action][Module]ReqDto` | e.g. `create-user-req.dto.ts` → `CreateUserReqDto` |
| `presentation/dto/res/[action]-[module]-res.dto.ts` | `[Action][Module]ResDto` | e.g. `create-user-res.dto.ts` → `CreateUserResDto` |

## Docs

| File/Folder | Contains |
|---|---|
| `docs/swagger/[module].swagger.ts` | Swagger decorator objects consumed by `@Api*` decorators in the controller |
| `docs/markdown/[flow-name].md` | Narrative documentation of a use case or full module flow |

## Shared / transversal (full projects)

| File | Class | Notes |
|---|---|---|
| `shared/decorators/[Name].ts` | — | PascalCase filename for decorators, e.g. `Public.ts`, `CurrentUser.ts`, `Roles.ts` |
| `shared/guards/[Name]Guard.ts` | `[Name]Guard` | e.g. `AuthGuard.ts` → `AuthGuard` |
| `shared/filters/[Name].filter.ts` | `[Name]Filter` | e.g. `ExceptionFormatter.filter.ts` → `ExceptionFormatterFilter` |
| `shared/interceptors/[Name].interceptor.ts` | `[Name]Interceptor` | e.g. `ResponseFormatter.interceptor.ts` → `ResponseFormatterInterceptor` |
| `shared/strategies/[Name].strategy.ts` | `[Name]Strategy` | e.g. `Jwt.strategy.ts` → `JwtStrategy` |
| `shared/errors/domain-error.base.ts` | `DomainError` | Abstract base class every domain error extends |
| `shared/types/[name].enum.ts` / `.interface.ts` | `[Name]` | e.g. `role.enum.ts` → `Role`, `jwt-payload.interface.ts` → `JwtPayload` |
| `shared/logger/logger.service.ts` | `LoggerService` | Injectable, `@Global()` module |
| `shared/database/database.module.ts` / `.provider.ts` | `DatabaseModule` | `@Global()` module |

## Tests

- `.http` files, named after the endpoint or flow they exercise: `tests/create-[module].http`, `tests/list-[module].http`.

---

## Quick cheat sheet

- Suffix tells you the role: `.module.ts`, `.controller.ts`, `.service.ts`, `.entity.ts`, `.vo.ts`, `.error.ts`, `.repository.ts`, `.repository.interface.ts`, `.queries.ts`, `.mapper.ts`, `.dto.ts`, `.guard.ts`, `.filter.ts`, `.interceptor.ts`, `.strategy.ts`, `.enum.ts`, `.interface.ts`, `.swagger.ts`.
- DTOs always split by direction (`dto/req/`, `dto/res/`) and by action (`create-`, `update-`, `list-`, …) — never a single generic `user.dto.ts` shared between request and response.
- A file's class name is always derivable from its path — if you need a comment to explain what a file is for, the name is wrong.
