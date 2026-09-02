# `@` Alias / `tsconfig.json` Paths

Every project this skill scaffolds configures the `@` alias to `./src` — no `../../` relative imports across module boundaries.

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

If the build/run setup needs runtime path resolution (e.g. `ts-node`, `tsc` output without a bundler that resolves `paths` itself), also register the resolver, for example with `tsconfig-paths`:

```json
// package.json (excerpt)
{
  "scripts": {
    "start:dev": "nest start --watch",
    "start:prod": "node -r tsconfig-paths/register dist/main.js"
  }
}
```

Import style throughout the project:

```typescript
// Correct
import { LoggerService } from '@/shared/logger/logger.service';

// Wrong — relative import crossing module boundaries
import { LoggerService } from '../../../shared/logger/logger.service';
```
