# Cross-Context Communication

When a module in one context needs data or behavior from a module in another context, these rules are non-negotiable:

1. **Only through public methods of the other module's `application/[module].service.ts`.** Never inject another context's repository or domain entities directly — that's the "Cross-context repository injection" anti-pattern.
2. **Public methods return that context's own DTOs, never domain entities.** The consuming context should never see the producing context's internal domain shape.
3. **Dependencies between contexts are unidirectional.** If you find two contexts calling each other in both directions to complete a flow ("Chatty Contexts"), that's a sign the boundary is wrong — either merge the contexts, or extract a third context that owns the shared responsibility.
4. **Atomic cross-context operations use a shared transaction**, propagated as an optional parameter through the public methods involved (a `QueryRunner` or the equivalent in the data access library). Example: recording a sale and decrementing stock must succeed or fail together.
5. **A read-only context (e.g. `catalog`) never accepts writes from outside itself.**

```typescript
// sales/modules/sale/application/sale.service.ts
export class SaleService {
  constructor(
    // Correct: injecting the OTHER context's public service, not its repository.
    private readonly catalogService: CatalogService,
  ) {}

  async createSale(dto: CreateSaleDto) {
    const product = await this.catalogService.getProductById(dto.productId); // returns CatalogService's own DTO
    // ...
  }
}
```
