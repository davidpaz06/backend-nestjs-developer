# Controller Conventions

1. **Method order**: group by HTTP verb first — GET, POST, PUT, PATCH, DELETE, in that order — then within a verb, order by relevance to the module's business flow.
2. **No raw Swagger decoration inline.** Define the documentation objects in `modules/[module]/docs/swagger/` and pass those objects into the `@Api*` decorators on the route. Keeps the controller readable and the docs reviewable on their own.
3. **Guards apply globally by default.** `AuthGuard` and `RoleGuard` are registered once in `main.ts`. Any route that doesn't need authentication or authorization must carry `@Public()` explicitly — there's no implicit opt-out.
4. **Rate limiting on every route**, via `@nestjs/throttler`'s decorator, applied by default even on routes the user didn't explicitly ask to protect.
5. **Controllers stay thin.** Business logic and query construction belong in `application/` and `infrastructure/` — a controller with either is the "Fat Controller" anti-pattern.
6. **Pagination is limit-offset by default** to keep MVP velocity up. Move to cursor-based pagination only once the project actually needs it (high-churn lists, deep pagination, consistency issues with offset).

```typescript
@Controller('products')
@UseGuards(AuthGuard, RoleGuard)
export class ProductController {
  constructor(private readonly productService: ProductService) {}

  @Get()
  @Public()
  @Throttle({ default: { limit: 20, ttl: 60000 } })
  @ApiListProducts() // object from docs/swagger/
  findAll(@Query() query: ListProductsReqDto) {
    return this.productService.findAll(query);
  }

  @Post()
  @Roles(Role.ADMIN)
  @Throttle({ default: { limit: 20, ttl: 60000 } })
  @ApiCreateProduct()
  create(@Body() dto: CreateProductReqDto) {
    return this.productService.create(dto);
  }
}
```
