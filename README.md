# backend-nestjs-developer

Skill for NestJS + PostgreSQL backend projects using tactical Domain-Driven Design. It provides workflows for advising, implementing, debugging, and scaffolding APIs while keeping bounded contexts, layering, errors, authentication, and infrastructure boundaries explicit.

## What it covers

- Simple and full DDD project structures with bounded contexts and four layers: `domain/`, `application/`, `infrastructure/`, and `presentation/`.
- NestJS bootstrap conventions: `/api/v1` prefix, Helmet, CORS, validation, response formatting, exception formatting, and throttling.
- PostgreSQL access through `resonance-components/datahandler`, with shared database configuration and repository/query boundaries.
- JWT authentication in `httpOnly` cookies and static RBAC by default, with guidance for dynamic RBAC when live revocation is required.
- Injectable global logging through `LoggerService` and `ConfigService`-based environment access.
- Domain errors translated centrally by `ExceptionFormatterFilter`; application services never throw `HttpException`.
- Swagger documentation separated into `docs/swagger/` objects and `docs/markdown/` flow documentation.
- Cross-context communication through public service methods and context-owned DTOs.
- Naming conventions and an anti-pattern checklist applied during implementation and debugging.

## Core conventions

- Use `pnpm` for all package management and scripts.
- Configure the `@` alias to `./src`; avoid deep relative imports.
- Apply limit-offset pagination by default.
- Rate-limit every endpoint with `@nestjs/throttler`.
- Keep controllers thin: no business logic or raw queries.
- Keep repositories and domain entities private to their bounded context.
- Confirm the directory tree with the user before scaffolding files.
- Database schema design, migrations strategy, index tuning, and frontend development are out of scope for this skill.

All conventions are defaults and can be overridden for a project or conversation.

## Installation

### Personal installation

```bash
git clone https://github.com/davidpaz06/backend-nestjs-developer.git ~/.claude/skills/backend-nestjs-developer
```

### Project-only installation

```bash
git submodule add https://github.com/davidpaz06/backend-nestjs-developer.git .claude/skills/backend-nestjs-developer
```

### From a downloaded `.zip` or `.skill` file

```bash
# A .skill file is a ZIP archive with a different extension.
mv backend-nestjs-developer.skill backend-nestjs-developer.zip
unzip backend-nestjs-developer.zip -d ~/.claude/skills/backend-nestjs-developer
```

PowerShell:

```powershell
Rename-Item -Path "backend-nestjs-developer.skill" -NewName "backend-nestjs-developer.zip"
Expand-Archive -Path "backend-nestjs-developer.zip" -DestinationPath "$HOME\.claude\skills\backend-nestjs-developer" -Force
```

## Updating

```bash
cd ~/.claude/skills/backend-nestjs-developer
git pull
```

## Repository structure

```text
backend-nestjs-developer/
|-- SKILL.md
|-- README.md
|-- CHANGELOG.md
`-- references/
    |-- alias-tsconfig.md
    |-- anti-patterns.md
    |-- auth-pattern.md
    |-- bootstrap.md
    |-- controller-conventions.md
    |-- cross-context.md
    |-- database-pattern.md
    |-- ddd-conventions.md
    |-- error-handling.md
    |-- logging-pattern.md
    |-- naming-rules.md
    |-- patterns.md
    |-- project-structure-full.md
    `-- project-structure-simple.md
```

Start with [`SKILL.md`](SKILL.md) for the complete workflows. Use [`references/patterns.md`](references/patterns.md) as the index for selecting a detailed pattern.

## Requirements

- Node.js compatible with the target NestJS version
- pnpm
- NestJS
- PostgreSQL
- `resonance-components/datahandler`
