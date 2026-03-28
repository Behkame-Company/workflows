# Unit Testing with AI

> Isolated component testing with AI assistance — writing focused, independent tests that verify individual units of behavior.

## What Makes a Good AI-Generated Unit Test

The best unit tests share these characteristics whether written by humans or AI:

- **Independent** — Each test runs in isolation with no shared state
- **Focused** — Tests exactly one behavior or code path
- **Descriptive names** — The test name explains what is being verified and under what conditions
- **AAA pattern** — Arrange, Act, Assert structure makes tests readable and maintainable

```typescript
// ✅ Good: Follows AAA pattern with descriptive name
describe('UserService', () => {
  it('should return null when user is not found by email', async () => {
    // Arrange
    const mockRepo = {
      findByEmail: jest.fn().mockResolvedValue(null),
    };
    const service = new UserService(mockRepo as any);

    // Act
    const result = await service.getUserByEmail('nobody@example.com');

    // Assert
    expect(result).toBeNull();
    expect(mockRepo.findByEmail).toHaveBeenCalledWith('nobody@example.com');
  });
});
```

```typescript
// ❌ Bad: Vague name, no structure, tests implementation details
it('works', async () => {
  const service = new UserService(mockRepo);
  await service.getUserByEmail('test@test.com');
  expect(mockRepo.findByEmail).toHaveBeenCalledTimes(1);
});
```

## Mocking Strategies for AI

AI tools generate better tests when you tell them exactly what to mock and how. Be explicit in your prompts:

### Prompt Template for Mocking

```
Write unit tests for UserService.createUser().

Mock these dependencies:
- UserRepository: mock findByEmail (returns null for new users, returns User for existing)
- EmailService: mock sendWelcomeEmail (returns Promise<void>)
- PasswordHasher: mock hash (returns 'hashed_password')

Do NOT mock:
- The validation logic inside createUser
- The User entity constructor

Use jest.fn() for all mocks. Inject mocks via constructor.
```

### Complete Example with Proper Mocking

```typescript
import { UserService } from './user-service';
import { UserRepository } from './user-repository';
import { EmailService } from './email-service';
import { PasswordHasher } from './password-hasher';

// Create typed mocks
const createMockRepo = (): jest.Mocked<UserRepository> => ({
  findByEmail: jest.fn(),
  save: jest.fn(),
  delete: jest.fn(),
});

const createMockEmailService = (): jest.Mocked<EmailService> => ({
  sendWelcomeEmail: jest.fn().mockResolvedValue(undefined),
  sendPasswordReset: jest.fn().mockResolvedValue(undefined),
});

const createMockHasher = (): jest.Mocked<PasswordHasher> => ({
  hash: jest.fn().mockResolvedValue('hashed_password'),
  verify: jest.fn().mockResolvedValue(true),
});

describe('UserService.createUser', () => {
  let service: UserService;
  let mockRepo: jest.Mocked<UserRepository>;
  let mockEmail: jest.Mocked<EmailService>;
  let mockHasher: jest.Mocked<PasswordHasher>;

  beforeEach(() => {
    // Fresh mocks for every test — no shared state
    mockRepo = createMockRepo();
    mockEmail = createMockEmailService();
    mockHasher = createMockHasher();
    service = new UserService(mockRepo, mockEmail, mockHasher);
  });

  it('should create user and send welcome email for new registrations', async () => {
    // Arrange
    mockRepo.findByEmail.mockResolvedValue(null);
    mockRepo.save.mockResolvedValue({ id: '1', email: 'new@example.com', name: 'New User' });

    // Act
    const result = await service.createUser({
      email: 'new@example.com',
      name: 'New User',
      password: 'securePass123!',
    });

    // Assert
    expect(result).toEqual({ id: '1', email: 'new@example.com', name: 'New User' });
    expect(mockHasher.hash).toHaveBeenCalledWith('securePass123!');
    expect(mockRepo.save).toHaveBeenCalledWith(
      expect.objectContaining({
        email: 'new@example.com',
        passwordHash: 'hashed_password',
      })
    );
    expect(mockEmail.sendWelcomeEmail).toHaveBeenCalledWith('new@example.com');
  });

  it('should throw ConflictError when email already exists', async () => {
    // Arrange
    mockRepo.findByEmail.mockResolvedValue({ id: '1', email: 'taken@example.com', name: 'Existing' });

    // Act & Assert
    await expect(
      service.createUser({ email: 'taken@example.com', name: 'Duplicate', password: 'pass123!' })
    ).rejects.toThrow('User with this email already exists');

    expect(mockRepo.save).not.toHaveBeenCalled();
    expect(mockEmail.sendWelcomeEmail).not.toHaveBeenCalled();
  });

  it('should throw ValidationError for invalid email format', async () => {
    await expect(
      service.createUser({ email: 'not-an-email', name: 'Bad Email', password: 'pass123!' })
    ).rejects.toThrow('Invalid email format');
  });
});
```

## Common AI Unit Test Issues

### 1. Testing Implementation Details

AI often tests _how_ something works instead of _what_ it does:

```typescript
// ❌ Testing implementation — breaks when you refactor
it('should call Array.filter then Array.map', () => {
  const filterSpy = jest.spyOn(Array.prototype, 'filter');
  const mapSpy = jest.spyOn(Array.prototype, 'map');
  getActiveUserNames(users);
  expect(filterSpy).toHaveBeenCalled();
  expect(mapSpy).toHaveBeenCalled();
});

// ✅ Testing behavior — survives refactoring
it('should return names of active users only', () => {
  const users = [
    { name: 'Alice', active: true },
    { name: 'Bob', active: false },
    { name: 'Carol', active: true },
  ];
  expect(getActiveUserNames(users)).toEqual(['Alice', 'Carol']);
});
```

### 2. Shared State Between Tests

AI sometimes creates shared variables that leak between tests:

```typescript
// ❌ Shared mutable state — test order matters
let users = [];

it('should add a user', () => {
  users.push({ name: 'Alice' });
  expect(users).toHaveLength(1);
});

it('should start empty', () => {
  // FAILS — users still has Alice from previous test
  expect(users).toHaveLength(0);
});

// ✅ Reset in beforeEach
let users: User[];

beforeEach(() => {
  users = [];
});
```

### 3. Brittle Assertions

AI-generated tests often make assertions that are too specific:

```typescript
// ❌ Brittle — breaks if timestamp format changes or ID generation changes
expect(result).toEqual({
  id: '550e8400-e29b-41d4-a716-446655440000',
  createdAt: '2024-01-15T10:30:00.000Z',
  name: 'Test User',
});

// ✅ Flexible — tests what matters
expect(result).toMatchObject({
  name: 'Test User',
});
expect(result.id).toBeDefined();
expect(result.createdAt).toBeInstanceOf(Date);
```

## Prompting Tips for Better Unit Tests

| Goal | Prompt Addition |
|------|----------------|
| Avoid implementation testing | "Test behavior and outputs, not internal method calls" |
| Get proper isolation | "Create fresh mocks in beforeEach, no shared mutable state" |
| Get meaningful assertions | "Include at least 2-3 assertions per test case" |
| Cover edge cases | "Include tests for null inputs, empty arrays, and error paths" |
| Get descriptive names | "Use the pattern: should [expected behavior] when [condition]" |

## Next Steps

- [Integration Testing](./integration-testing.md) — Test how units work together across boundaries
- [Test Quality Metrics](../05-coverage-and-quality/test-quality-metrics.md) — Measure whether your unit tests actually catch bugs
- [Mutation Testing](../05-coverage-and-quality/mutation-testing.md) — Validate that unit test assertions are strong enough
