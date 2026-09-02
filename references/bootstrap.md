# Application Bootstrap (`main.ts`)

Every project — simple or full — wires the same non-negotiables into `main.ts`. This isn't its own numbered pattern in the original spec, but it's where all of them come together, so treat it as the checklist to verify before calling a project "bootstrapped":

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import helmet from 'helmet';
import { AppModule } from './app/app.module';
import { ResponseFormatterInterceptor } from './shared/interceptors/ResponseFormatter.interceptor';
import { ExceptionFormatterFilter } from './shared/filters/ExceptionFormatter.filter';
import { ValidationPipe } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const config = app.get(ConfigService);

  app.setGlobalPrefix('api/v1');

  app.use(helmet());
  app.enableCors({
    origin: config.get('CORS_ORIGIN', 'http://localhost:3000'),
    credentials: true,
    methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
    allowedHeaders: 'Content-Type, Accept, Authorization',
  });

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  app.useGlobalInterceptors(new ResponseFormatterInterceptor());
  app.useGlobalFilters(new ExceptionFormatterFilter());

  // app.useLogger(app.get(LoggerService)); — see references/logging-pattern.md
  // Swagger setup (SwaggerModule.setup) goes here for full projects.

  await app.listen(config.get('PORT', 3000));
}
bootstrap();
```

Success response shape (from `ResponseFormatterInterceptor`):

```typescript
{
  statusCode: number;
  message: string;
  data: T;
  timestamp: string;
}
```

Error response shape (from `ExceptionFormatterFilter`, see `references/error-handling.md` for the full filter implementation):

```typescript
{
  statusCode: number;
  timestamp: string;
  path: string;
  method: string;
  message: string;
  error: string;
}
```

Guards (`AuthGuard`, `RoleGuard`) are registered globally, also in `main.ts` — see `references/auth-pattern.md`. Any route that doesn't require auth must be marked explicitly with `@Public()`.
