# Integration Testing with AI

> Cross-boundary testing patterns — verifying that components work together correctly, especially critical when AI generates code with mocked unit tests that can hide integration failures.

## Why Integration Tests Matter for AI Code

Unit tests with mocks tell you that each piece works _in isolation_. Integration tests tell you the pieces actually fit together. This gap is especially dangerous with AI-generated code:

```
AI generates UserService with mocked UserRepository
  → Unit tests pass ✅
  → But UserService calls repo.findByEmail(email)
  → Real UserRepository.findByEmail expects { where: { email } }
  → Production breaks 💥
```

**The mock matched the AI's _assumption_ about the interface, not the real interface.** Integration tests catch this class of bugs that unit tests with mocks structurally cannot.

### The Integration Testing Principle

> If you mocked it in a unit test, you need an integration test where it's real.

## What to Test at Integration Level

| Boundary | What to Verify |
|----------|---------------|
| Service → Database | Queries return expected data, migrations work, constraints enforced |
| Service → Service | Correct data passed between services, error propagation works |
| API → Service | Request parsing, validation, response formatting, status codes |
| Service → External API | Request format, auth headers, response parsing, error handling |

## Database Integration Testing

Use test containers to spin up real databases for integration tests. This catches SQL errors, constraint violations, and ORM misconfigurations that mocked repositories hide.

### Example: Database Tests with Testcontainers

```typescript
import { PostgreSqlContainer, StartedPostgreSqlContainer } from '@testcontainers/postgresql';
import { DataSource } from 'typeorm';
import { UserRepository } from './user-repository';
import { User } from './user.entity';

describe('UserRepository (integration)', () => {
  let container: StartedPostgreSqlContainer;
  let dataSource: DataSource;
  let repository: UserRepository;

  beforeAll(async () => {
    // Start a real PostgreSQL instance
    container = await new PostgreSqlContainer()
      .withDatabase('testdb')
      .start();

    dataSource = new DataSource({
      type: 'postgres',
      host: container.getHost(),
      port: container.getMappedPort(5432),
      username: container.getUsername(),
      password: container.getPassword(),
      database: container.getDatabase(),
      entities: [User],
      synchronize: true,
    });

    await dataSource.initialize();
    repository = new UserRepository(dataSource);
  }, 60000);

  afterAll(async () => {
    await dataSource.destroy();
    await container.stop();
  });

  beforeEach(async () => {
    // Clean slate for each test
    await dataSource.getRepository(User).clear();
  });

  it('should save and retrieve a user by email', async () => {
    // Arrange
    const user = { email: 'alice@example.com', name: 'Alice', passwordHash: 'hash123' };

    // Act
    const saved = await repository.create(user);
    const found = await repository.findByEmail('alice@example.com');

    // Assert
    expect(found).toBeDefined();
    expect(found!.id).toBe(saved.id);
    expect(found!.email).toBe('alice@example.com');
    expect(found!.name).toBe('Alice');
  });

  it('should enforce unique email constraint', async () => {
    await repository.create({ email: 'dup@example.com', name: 'First', passwordHash: 'hash1' });

    await expect(
      repository.create({ email: 'dup@example.com', name: 'Second', passwordHash: 'hash2' })
    ).rejects.toThrow();
  });

  it('should return null for non-existent email', async () => {
    const result = await repository.findByEmail('nobody@example.com');
    expect(result).toBeNull();
  });
});
```

## API Integration Testing

Test HTTP endpoints with real service wiring. Use `supertest` (Node.js) to make requests against your app without starting a network server.

### Example: API Tests with Supertest

```typescript
import request from 'supertest';
import { createApp } from './app';
import { setupTestDatabase, teardownTestDatabase } from './test-helpers';

describe('POST /api/users', () => {
  let app: Express;
  let db: TestDatabase;

  beforeAll(async () => {
    db = await setupTestDatabase();
    app = createApp({ database: db.dataSource });
  });

  afterAll(async () => {
    await teardownTestDatabase(db);
  });

  it('should create a user and return 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'new@example.com', name: 'New User', password: 'Str0ng!Pass' })
      .expect(201);

    expect(response.body).toMatchObject({
      email: 'new@example.com',
      name: 'New User',
    });
    expect(response.body).toHaveProperty('id');
    expect(response.body).not.toHaveProperty('password');
    expect(response.body).not.toHaveProperty('passwordHash');
  });

  it('should return 409 for duplicate email', async () => {
    await request(app)
      .post('/api/users')
      .send({ email: 'dup@example.com', name: 'First', password: 'Str0ng!Pass' });

    const response = await request(app)
      .post('/api/users')
      .send({ email: 'dup@example.com', name: 'Second', password: 'Str0ng!Pass' })
      .expect(409);

    expect(response.body.error).toMatch(/already exists/i);
  });

  it('should return 400 for invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'not-valid', name: 'Bad', password: 'Str0ng!Pass' })
      .expect(400);

    expect(response.body.errors).toContainEqual(
      expect.objectContaining({ field: 'email' })
    );
  });
});
```

## Service-to-Service Testing

When services communicate over HTTP, message queues, or gRPC, test the actual communication:

```typescript
describe('OrderService → InventoryService integration', () => {
  let orderService: OrderService;
  let inventoryService: InventoryService;
  let inventoryServer: http.Server;

  beforeAll(async () => {
    // Start real inventory service on test port
    inventoryService = new InventoryService(testDb);
    inventoryServer = await inventoryService.listen(0); // random port
    const port = (inventoryServer.address() as AddressInfo).port;

    // Order service points at real inventory
    orderService = new OrderService({
      inventoryUrl: `http://localhost:${port}`,
      database: testDb,
    });
  });

  afterAll(async () => {
    inventoryServer.close();
  });

  it('should decrement inventory when order is placed', async () => {
    // Arrange — seed inventory
    await inventoryService.setStock('WIDGET-1', 10);

    // Act — place order through order service
    const order = await orderService.placeOrder({
      items: [{ sku: 'WIDGET-1', quantity: 3 }],
      customerId: 'cust-1',
    });

    // Assert — verify both services reflect the change
    expect(order.status).toBe('confirmed');
    const stock = await inventoryService.getStock('WIDGET-1');
    expect(stock).toBe(7);
  });

  it('should reject order when insufficient stock', async () => {
    await inventoryService.setStock('WIDGET-2', 2);

    await expect(
      orderService.placeOrder({
        items: [{ sku: 'WIDGET-2', quantity: 5 }],
        customerId: 'cust-1',
      })
    ).rejects.toThrow('Insufficient stock');

    // Verify inventory unchanged
    const stock = await inventoryService.getStock('WIDGET-2');
    expect(stock).toBe(2);
  });
});
```

## Setup and Teardown Patterns

### Layered Setup

```typescript
// Global setup — runs once for entire test suite
beforeAll(async () => {
  await startTestContainers();
  await runMigrations();
});

// Per-describe setup — runs once per describe block
beforeAll(async () => {
  testUser = await createTestUser();
  authToken = await getAuthToken(testUser);
});

// Per-test setup — runs before each individual test
beforeEach(async () => {
  await cleanTables(['orders', 'order_items']);
});

// Teardown in reverse order
afterAll(async () => {
  await stopTestContainers();
});
```

### Test Data Factories

```typescript
// Create reusable test data factories for integration tests
const testFactory = {
  user: (overrides = {}) => ({
    email: `user-${randomUUID()}@test.com`,
    name: 'Test User',
    password: 'TestPass123!',
    ...overrides,
  }),

  order: (userId: string, overrides = {}) => ({
    customerId: userId,
    items: [{ sku: 'TEST-ITEM', quantity: 1 }],
    ...overrides,
  }),
};
```

## Prompting AI for Integration Tests

AI needs more context for integration tests than unit tests. Specify clearly what stays real and what gets mocked:

### Prompt Template

```
Write integration tests for the order placement workflow.

System under test: OrderController → OrderService → OrderRepository → PostgreSQL

Keep REAL (do not mock):
- OrderService business logic
- OrderRepository database queries
- Database (use testcontainers PostgreSQL)

Mock ONLY:
- EmailService (external dependency, mock with jest.fn())
- PaymentGateway (external API, mock with nock or jest.fn())

Boundaries to test:
1. HTTP request → correct database state
2. Validation errors return proper HTTP status codes
3. Database constraints are enforced (no duplicate orders)
4. Transaction rollback on payment failure

Use beforeAll for container setup, beforeEach for data cleanup.
Include both happy path and error scenarios.
```
