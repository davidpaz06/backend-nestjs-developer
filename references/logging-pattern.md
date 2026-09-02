# Logging

The logger is an **injectable service**, never instantiated directly with `new Logger()`. This is what lets you swap NestJS's internal logger in `main.ts` and inject the same logger anywhere via DI without re-importing a module.

```typescript
// shared/logger/logger.module.ts
@Global()
@Module({
  providers: [LoggerService],
  exports: [LoggerService],
})
export class LoggerModule {}
```

```typescript
// shared/logger/logger.service.ts
@Injectable()
export class LoggerService {
  private readonly logger = new Logger();

  log(message: string, context?: string) {
    this.logger.log(message, context);
  }
  error(message: string, trace?: string, context?: string) {
    this.logger.error(message, trace, context);
  }
  warn(message: string, context?: string) {
    this.logger.warn(message, context);
  }
  debug(message: string, context?: string) {
    this.logger.debug(message, context);
  }
}
```

`LoggerModule` is `@Global()` and imported once, in `app.module.ts` — same as `DatabaseModule`. In `main.ts`, replace Nest's internal logger with it: `app.useLogger(app.get(LoggerService))`.

Usage in any service:

```typescript
constructor(private readonly logger: LoggerService) {}

this.logger.log('Creating user', UserService.name);
```

Never swallow an exception without logging it or re-throwing — see the "Swallowed exception" anti-pattern in `references/anti-patterns.md`.
