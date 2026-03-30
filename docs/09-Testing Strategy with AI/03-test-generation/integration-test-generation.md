# Integration Test Generation

> AI-assisted integration testing across boundaries — because unit tests with mocks can pass while the real system fails.

---

## Why AI-Generated Code Needs Integration Tests

Unit tests with mocks verify that your code calls dependencies correctly **assuming the mock is accurate**. But AI-generated code often has subtle integration issues:

```
Unit test with mock:     ✅ PASS (mock returns what you told it to)
Real integration test:   ❌ FAIL (actual database returns different shape)
```

| Problem | Unit Test | Integration Test |
|---------|-----------|-----------------|
| Wrong SQL syntax | ❌ Mock doesn't run SQL | ✅ Catches at DB level |
| API returns different shape | ❌ Mock returns expected shape | ✅ Catches real response |
| Missing middleware | ❌ No middleware in mock | ✅ Full HTTP pipeline runs |
| Transaction isolation issues | ❌ No real transactions | ✅ Real DB transactions |
| Environment configuration | ❌ Hardcoded mock values | ✅ Uses real config |

**Rule of thumb:** If AI wrote the code, write at least one integration test that exercises the real dependency.

---

## Integration Test Categories

```
┌─────────────────────────────────────────────────────────────┐
│                  Integration Test Spectrum                    │
│                                                              │
│  Unit          Component        Service          System      │
│  (mocked)      (real DB)        (real APIs)      (full E2E)  │
│  ◄──────────────────────────────────────────────────────────▶│
│  Fast/Isolated                              Slow/Realistic   │
│                                                              │
│  This document focuses on Component and Service levels       │
└─────────────────────────────────────────────────────────────┘
```

---

## Pattern 1: Database Integration Tests

### Prompt Template

```
Generate integration tests for this repository function that uses a real
database connection. Include proper setup/teardown.

```typescript
{pasteRepositoryCode}
```

Requirements:
- Use a real test database (not mocks)
- Set up test data in beforeEach, clean up in afterEach
- Test actual SQL queries execute correctly
- Verify data persistence (write then read back)
- Test constraint violations (unique, foreign key)
- Use transactions for test isolation

Use Jest with {databaseLibrary}.
```

### Example: Database Integration Test

**Function under test:**

```typescript
// user-repository.ts
import { Pool } from 'pg';

export class UserRepository {
  constructor(private pool: Pool) {}

  async create(email: string, name: string): Promise<User> {
    const result = await this.pool.query(
      'INSERT INTO users (email, name) VALUES ($1, $2) RETURNING *',
      [email, name]
    );
    return result.rows[0];
  }

  async findByEmail(email: string): Promise<User | null> {
    const result = await this.pool.query(
      'SELECT * FROM users WHERE email = $1',
      [email]
    );
    return result.rows[0] || null;
  }
}
```

**Generated Integration Tests:**

```typescript
// user-repository.integration.test.ts
import { Pool } from 'pg';
import { UserRepository } from './user-repository';

describe('UserRepository (integration)', () => {
  let pool: Pool;
  let repo: UserRepository;

  beforeAll(async () => {
    pool = new Pool({
      connectionString: process.env.TEST_DATABASE_URL,
    });
    repo = new UserRepository(pool);
  });

  afterAll(async () => {
    await pool.end();
  });

  beforeEach(async () => {
    // Clean slate for each test
    await pool.query('DELETE FROM users');
  });

  describe('create', () => {
    it('should insert a user and return it with an id', async () => {
      const user = await repo.create('test@example.com', 'Test User');

      expect(user.id).toBeDefined();
      expect(user.email).toBe('test@example.com');
      expect(user.name).toBe('Test User');
    });

    it('should persist the user in the database', async () => {
      await repo.create('persist@example.com', 'Persisted');

      // Verify with a raw query — not through the repository
      const result = await pool.query(
        "SELECT * FROM users WHERE email = 'persist@example.com'"
      );
      expect(result.rows).toHaveLength(1);
      expect(result.rows[0].name).toBe('Persisted');
    });

    it('should reject duplicate emails', async () => {
      await repo.create('dup@example.com', 'First');

      await expect(
        repo.create('dup@example.com', 'Second')
      ).rejects.toThrow(/unique/i);
    });

    it('should handle special characters in name', async () => {
      const user = await repo.create('special@example.com', "O'Brien-José");

      const found = await repo.findByEmail('special@example.com');
      expect(found?.name).toBe("O'Brien-José");
    });
  });

  describe('findByEmail', () => {
    it('should return the user when found', async () => {
      await repo.create('find@example.com', 'Findable');

      const user = await repo.findByEmail('find@example.com');
      expect(user).not.toBeNull();
      expect(user?.name).toBe('Findable');
    });

    it('should return null when user does not exist', async () => {
      const user = await repo.findByEmail('nonexistent@example.com');
      expect(user).toBeNull();
    });

    it('should be case-sensitive for email lookup', async () => {
      await repo.create('Case@Example.com', 'Case Test');

      const found = await repo.findByEmail('case@example.com');
      // Behavior depends on DB collation — this test documents it
      expect(found).toBeNull(); // or not null, depending on collation
    });
  });
});
```

---

## Pattern 2: API Integration Tests

### Prompt Template

```
Generate integration tests for this API endpoint using supertest.
Test the full HTTP pipeline including middleware, validation, and response format.

```typescript
{pasteRouteHandler}
```

Test:
- Correct HTTP status codes
- Response body shape and content
- Request validation (missing fields, wrong types)
- Authentication/authorization if applicable
- Content-Type headers

Use Jest with supertest.
```

### Example: API Integration Test

```typescript
// products-api.integration.test.ts
import request from 'supertest';
import { app } from './app';
import { db } from './database';

describe('Products API (integration)', () => {
  beforeAll(async () => {
    await db.migrate.latest();
  });

  afterAll(async () => {
    await db.destroy();
  });

  beforeEach(async () => {
    await db('products').truncate();
    await db('products').insert([
      { id: '1', name: 'Widget', price: 9.99, category: 'tools' },
      { id: '2', name: 'Gadget', price: 24.99, category: 'electronics' },
      { id: '3', name: 'Sprocket', price: 4.99, category: 'tools' },
    ]);
  });

  describe('GET /api/products', () => {
    it('should return all products with 200 status', async () => {
      const res = await request(app).get('/api/products');

      expect(res.status).toBe(200);
      expect(res.headers['content-type']).toMatch(/json/);
      expect(res.body.items).toHaveLength(3);
    });

    it('should filter by category', async () => {
      const res = await request(app).get('/api/products?category=tools');

      expect(res.status).toBe(200);
      expect(res.body.items).toHaveLength(2);
      expect(res.body.items.every((p: any) => p.category === 'tools')).toBe(true);
    });

    it('should paginate with limit and offset', async () => {
      const res = await request(app).get('/api/products?limit=1&offset=1');

      expect(res.body.items).toHaveLength(1);
      expect(res.body.total).toBe(3);
    });
  });

  describe('POST /api/products', () => {
    it('should create a product and return 201', async () => {
      const newProduct = {
        name: 'New Item',
        price: 15.99,
        category: 'misc',
      };

      const res = await request(app)
        .post('/api/products')
        .send(newProduct)
        .set('Content-Type', 'application/json');

      expect(res.status).toBe(201);
      expect(res.body.id).toBeDefined();
      expect(res.body.name).toBe('New Item');

      // Verify persistence
      const getRes = await request(app).get(`/api/products/${res.body.id}`);
      expect(getRes.status).toBe(200);
      expect(getRes.body.name).toBe('New Item');
    });

    it('should return 400 for missing required fields', async () => {
      const res = await request(app)
        .post('/api/products')
        .send({ name: 'No Price' })
        .set('Content-Type', 'application/json');

      expect(res.status).toBe(400);
      expect(res.body.error).toBeDefined();
    });

    it('should return 400 for invalid price', async () => {
      const res = await request(app)
        .post('/api/products')
        .send({ name: 'Bad', price: -5, category: 'x' })
        .set('Content-Type', 'application/json');

      expect(res.status).toBe(400);
    });
  });
});
```

---

## Pattern 3: Cross-Service Integration

When your code interacts with external services, test with real (test) instances or official test containers:

```typescript
// payment-integration.test.ts
import { PaymentProcessor } from './payment-processor';
import { StripeTestClient } from './test-helpers/stripe';

describe('PaymentProcessor (integration with Stripe test mode)', () => {
  let processor: PaymentProcessor;

  beforeAll(() => {
    processor = new PaymentProcessor({
      apiKey: process.env.STRIPE_TEST_KEY!,
    });
  });

  it('should process a successful payment', async () => {
    const result = await processor.charge({
      amount: 1000, // $10.00
      currency: 'usd',
      token: 'tok_visa', // Stripe test token
    });

    expect(result.success).toBe(true);
    expect(result.chargeId).toMatch(/^ch_/);
  });

  it('should handle declined cards', async () => {
    const result = await processor.charge({
      amount: 1000,
      currency: 'usd',
      token: 'tok_chargeDeclined', // Stripe test token for decline
    });

    expect(result.success).toBe(false);
    expect(result.error).toContain('declined');
  });

  it('should handle insufficient funds', async () => {
    const result = await processor.charge({
      amount: 1000,
      currency: 'usd',
      token: 'tok_chargeDeclinedInsufficientFunds',
    });

    expect(result.success).toBe(false);
    expect(result.error).toContain('insufficient');
  });
});
```

---

## When to Use Integration vs Unit Tests

| Signal | Use Unit Tests | Use Integration Tests |
|--------|---------------|----------------------|
| Pure logic / calculations | ✅ | |
| Data transformation | ✅ | |
| Database queries | | ✅ |
| API endpoints | | ✅ |
| File system operations | | ✅ |
| Cross-module coordination | | ✅ |
| AI-generated data access code | | ✅ (mocks can mask wrong SQL) |
| AI-generated API handlers | | ✅ (mocks can mask wrong status codes) |
| AI-generated business logic | ✅ | ⚠️ Also if logic spans modules |

---

## Setup/Teardown Best Practices

```typescript
// Common integration test patterns

// 1. Database cleanup — truncate is faster than delete
beforeEach(async () => {
  await db.raw('TRUNCATE TABLE users, orders, products CASCADE');
});

// 2. Transaction rollback — fastest, guaranteed clean state
beforeEach(async () => {
  await db.raw('BEGIN');
});
afterEach(async () => {
  await db.raw('ROLLBACK');
});

// 3. Test containers — spin up fresh DB per suite
beforeAll(async () => {
  container = await new GenericContainer('postgres:15')
    .withEnvironment({ POSTGRES_PASSWORD: 'test' })
    .withExposedPorts(5432)
    .start();
});
afterAll(async () => {
  await container.stop();
});
```

---

## Prompting Tips for Integration Tests

- **Specify the real dependencies** — "Use a real PostgreSQL connection, not mocks"
- **Request setup/teardown** — "Include beforeEach cleanup and afterAll connection close"
- **Ask for persistence verification** — "After creating, verify with a separate query"
- **Name the test file clearly** — `*.integration.test.ts` separates from unit tests
- **Request realistic test data** — "Use realistic field values, not 'test' and 'foo'"
