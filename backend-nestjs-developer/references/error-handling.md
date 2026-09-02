# Error Handling

Two kinds of error, two different jobs:

## Domain errors

Live in `contexts/[context]/modules/[module]/domain/errors/`. They represent business rule violations, not HTTP concerns, and extend a shared base:

```typescript
// shared/errors/domain-error.base.ts
export abstract class DomainError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
}
```

```typescript
// contexts/sales/modules/sale/domain/errors/insufficient-stock.error.ts
export class InsufficientStockError extends DomainError {
  readonly code = 'INSUFFICIENT_STOCK';
  readonly statusCode = 409;

  constructor(productId: string) {
    super(`Insufficient stock for product ${productId}`);
  }
}
```

**Rule: `application/` services throw domain errors — `throw new InsufficientStockError(id)` — never `HttpException` directly.** Throwing HTTP exceptions from application or domain code is an anti-pattern (see "Throw HttpException from application/"); the filter below is the only place that knows about HTTP status codes for business errors.

## HTTP errors

Whatever NestJS or a library throws on its own — `ValidationPipe` failures, guard rejections, uncaught exceptions. Captured the same way as domain errors.

## Centralized capture

`ExceptionFormatterFilter` in `shared/filters/` catches both kinds and responds with the standard error shape (see `references/bootstrap.md`), picking the right status code per error type:

```typescript
@Catch()
export class ExceptionFormatterFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    const isDomainError = exception instanceof DomainError;
    const status = isDomainError
      ? exception.statusCode
      : exception instanceof HttpException
        ? exception.getStatus()
        : 500;

    const message =
      (exception as any)?.response?.message ??
      (exception as any)?.message ??
      status;

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message,
      error: isDomainError ? (exception as DomainError).code : undefined,
    });
  }
}
```

## Transactions

- Don't leave multiple related writes without a shared transaction (`QueryRunner` or the data access library's equivalent) and a rollback path — see "Implicit or absent transaction."
- Never call an external service (email, payment gateway) from inside a database transaction block — see "Long-running transaction with external I/O." Do the external call before or after the transactional block, not inside it.
