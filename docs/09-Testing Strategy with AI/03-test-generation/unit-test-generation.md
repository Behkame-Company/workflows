# Unit Test Generation

> Generating comprehensive unit tests with AI — from happy paths to edge cases, with mocking strategies and quality tips.

---

## What Makes a Good AI-Generated Unit Test

AI-generated unit tests must meet the same quality bar as human-written tests:

| Quality Dimension | What to Verify |
|-------------------|---------------|
| **Isolation** | Tests only the unit, not its dependencies |
| **Determinism** | Same result every run — no randomness, no time dependency |
| **Readability** | Arrange-Act-Assert structure, descriptive names |
| **Completeness** | Happy path + validation + error + edge cases |
| **Independence** | No shared mutable state between tests |
| **Speed** | Runs in milliseconds, not seconds |

---

## The Arrange-Act-Assert Pattern

Always instruct AI to follow AAA structure. This produces consistent, readable tests.

```typescript
it('should calculate compound interest correctly', () => {
  // Arrange — set up the test data
  const principal = 1000;
  const rate = 0.05;
  const years = 10;

  // Act — call the function under test
  const result = calculateCompoundInterest(principal, rate, years);

  // Assert — verify the result
  expect(result).toBeCloseTo(1628.89, 2);
});
```

**Prompt tip:** Add "Use the Arrange-Act-Assert pattern for each test" to your prompts.

---

## Comprehensive Prompting for Unit Tests

### The Prompt Structure

```
Generate unit tests for the following function.
Use Jest with TypeScript. Follow the Arrange-Act-Assert pattern.

```typescript
{pasteFunction}
```

Cover these categories:
1. **Happy path**: Typical valid inputs and expected outputs
2. **Validation**: Invalid inputs, missing parameters, wrong types
3. **Error handling**: Error conditions and thrown exceptions
4. **Edge cases**: Boundary values, empty inputs, large inputs
5. **Return type**: Verify the shape and type of returned values

Mock external dependencies using Jest mocks. Do not test dependencies
themselves — only test the function's behavior.
```

---

## Complete Example

### Input Function

```typescript
// order-calculator.ts
import { TaxService } from './tax-service';
import { InventoryService } from './inventory-service';

interface OrderItem {
  productId: string;
  quantity: number;
  unitPrice: number;
}

interface OrderSummary {
  subtotal: number;
  tax: number;
  total: number;
  items: Array<OrderItem & { available: boolean }>;
}

export async function calculateOrder(
  items: OrderItem[],
  taxService: TaxService,
  inventoryService: InventoryService,
  state: string
): Promise<OrderSummary> {
  if (!items.length) throw new Error('Order must have at least one item');

  const subtotal = items.reduce(
    (sum, item) => sum + item.quantity * item.unitPrice,
    0
  );

  const taxRate = await taxService.getTaxRate(state);
  const tax = Math.round(subtotal * taxRate * 100) / 100;

  const itemsWithAvailability = await Promise.all(
    items.map(async (item) => ({
      ...item,
      available: await inventoryService.checkAvailability(
        item.productId,
        item.quantity
      ),
    }))
  );

  return {
    subtotal,
    tax,
    total: subtotal + tax,
    items: itemsWithAvailability,
  };
}
```

### Generated Unit Tests

```typescript
// order-calculator.test.ts
import { calculateOrder } from './order-calculator';
import { TaxService } from './tax-service';
import { InventoryService } from './inventory-service';

// Mock dependencies
const mockTaxService: jest.Mocked<TaxService> = {
  getTaxRate: jest.fn(),
};

const mockInventoryService: jest.Mocked<InventoryService> = {
  checkAvailability: jest.fn(),
};

describe('calculateOrder', () => {
  beforeEach(() => {
    jest.clearAllMocks();
    mockTaxService.getTaxRate.mockResolvedValue(0.08); // 8% default
    mockInventoryService.checkAvailability.mockResolvedValue(true);
  });

  // ── Happy Path ──────────────────────────────────────

  it('should calculate order with single item', async () => {
    // Arrange
    const items = [{ productId: 'P1', quantity: 2, unitPrice: 25.0 }];

    // Act
    const result = await calculateOrder(
      items,
      mockTaxService,
      mockInventoryService,
      'CA'
    );

    // Assert
    expect(result.subtotal).toBe(50.0);
    expect(result.tax).toBe(4.0);
    expect(result.total).toBe(54.0);
  });

  it('should calculate order with multiple items', async () => {
    const items = [
      { productId: 'P1', quantity: 1, unitPrice: 10.0 },
      { productId: 'P2', quantity: 3, unitPrice: 20.0 },
    ];

    const result = await calculateOrder(
      items,
      mockTaxService,
      mockInventoryService,
      'CA'
    );

    expect(result.subtotal).toBe(70.0);
    expect(result.tax).toBe(5.6);
    expect(result.total).toBe(75.6);
  });

  it('should include availability for each item', async () => {
    mockInventoryService.checkAvailability
      .mockResolvedValueOnce(true)
      .mockResolvedValueOnce(false);

    const items = [
      { productId: 'P1', quantity: 1, unitPrice: 10.0 },
      { productId: 'P2', quantity: 100, unitPrice: 5.0 },
    ];

    const result = await calculateOrder(
      items,
      mockTaxService,
      mockInventoryService,
      'CA'
    );

    expect(result.items[0].available).toBe(true);
    expect(result.items[1].available).toBe(false);
  });

  // ── Validation ──────────────────────────────────────

  it('should throw for empty items array', async () => {
    await expect(
      calculateOrder([], mockTaxService, mockInventoryService, 'CA')
    ).rejects.toThrow('Order must have at least one item');
  });

  // ── Dependency Interaction ──────────────────────────

  it('should call tax service with the correct state', async () => {
    const items = [{ productId: 'P1', quantity: 1, unitPrice: 10.0 }];

    await calculateOrder(items, mockTaxService, mockInventoryService, 'NY');

    expect(mockTaxService.getTaxRate).toHaveBeenCalledWith('NY');
    expect(mockTaxService.getTaxRate).toHaveBeenCalledTimes(1);
  });

  it('should check inventory for each item with correct quantity', async () => {
    const items = [
      { productId: 'P1', quantity: 5, unitPrice: 10.0 },
      { productId: 'P2', quantity: 3, unitPrice: 20.0 },
    ];

    await calculateOrder(items, mockTaxService, mockInventoryService, 'CA');

    expect(mockInventoryService.checkAvailability).toHaveBeenCalledWith('P1', 5);
    expect(mockInventoryService.checkAvailability).toHaveBeenCalledWith('P2', 3);
    expect(mockInventoryService.checkAvailability).toHaveBeenCalledTimes(2);
  });

  // ── Error Handling ──────────────────────────────────

  it('should propagate tax service errors', async () => {
    mockTaxService.getTaxRate.mockRejectedValue(new Error('Service unavailable'));

    const items = [{ productId: 'P1', quantity: 1, unitPrice: 10.0 }];

    await expect(
      calculateOrder(items, mockTaxService, mockInventoryService, 'CA')
    ).rejects.toThrow('Service unavailable');
  });

  it('should propagate inventory service errors', async () => {
    mockInventoryService.checkAvailability.mockRejectedValue(
      new Error('Inventory check failed')
    );

    const items = [{ productId: 'P1', quantity: 1, unitPrice: 10.0 }];

    await expect(
      calculateOrder(items, mockTaxService, mockInventoryService, 'CA')
    ).rejects.toThrow('Inventory check failed');
  });

  // ── Edge Cases ──────────────────────────────────────

  it('should handle zero tax rate', async () => {
    mockTaxService.getTaxRate.mockResolvedValue(0);

    const items = [{ productId: 'P1', quantity: 1, unitPrice: 100.0 }];
    const result = await calculateOrder(
      items,
      mockTaxService,
      mockInventoryService,
      'OR'
    );

    expect(result.tax).toBe(0);
    expect(result.total).toBe(100.0);
  });

  it('should round tax to two decimal places', async () => {
    mockTaxService.getTaxRate.mockResolvedValue(0.0725);

    const items = [{ productId: 'P1', quantity: 1, unitPrice: 33.33 }];
    const result = await calculateOrder(
      items,
      mockTaxService,
      mockInventoryService,
      'CA'
    );

    expect(result.tax).toBe(2.42); // 33.33 * 0.0725 = 2.416425 → 2.42
  });

  // ── Return Type Verification ────────────────────────

  it('should return the correct shape', async () => {
    const items = [{ productId: 'P1', quantity: 1, unitPrice: 10.0 }];
    const result = await calculateOrder(
      items,
      mockTaxService,
      mockInventoryService,
      'CA'
    );

    expect(result).toEqual({
      subtotal: expect.any(Number),
      tax: expect.any(Number),
      total: expect.any(Number),
      items: expect.arrayContaining([
        expect.objectContaining({
          productId: expect.any(String),
          quantity: expect.any(Number),
          unitPrice: expect.any(Number),
          available: expect.any(Boolean),
        }),
      ]),
    });
  });
});
```

---

## Mocking Strategy Guidance

Tell the AI exactly how to handle dependencies:

```
Mocking rules:
- Mock all external services (database, API, file system)
- Mock time-dependent functions (Date.now, setTimeout)
- Do NOT mock the function under test
- Do NOT mock pure utility functions (math, string formatting)
- Use jest.fn() for simple mocks
- Use jest.spyOn() when you need to verify calls AND keep behavior
- Reset all mocks in beforeEach
```

### Mock Decision Table

| Dependency Type | Mock? | Reason |
|----------------|-------|--------|
| Database queries | ✅ Yes | Isolation, speed |
| HTTP/API calls | ✅ Yes | Isolation, reliability |
| File system | ✅ Yes | Isolation, portability |
| Date/time | ✅ Yes | Determinism |
| Pure functions in same module | ❌ No | They're part of the unit |
| Configuration values | ⚠️ Sometimes | Mock if they change behavior significantly |

---

## Tips for Better Unit Test Quality

1. **Name tests as behavior descriptions** — `it('should throw when email is empty')` not `it('test case 3')`
2. **One assertion concept per test** — multiple `expect()` calls are fine if they verify one behavior
3. **Provide sample data in the prompt** — AI generates more realistic tests with realistic data
4. **Ask for negative tests explicitly** — AI tends to generate happy paths unless you ask for errors
5. **Specify the mock strategy** — without guidance, AI may over-mock or under-mock
6. **Request boundary value analysis** — "test at exactly the boundary, one below, and one above"
