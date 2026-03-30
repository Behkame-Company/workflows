# Spec-Driven Testing

> Generating tests from formal specifications — turning API specs, type definitions, and requirements documents into comprehensive test suites.

---

## The Spec → Test → Implementation Pipeline

Formal specifications are the highest-value input for AI test generation. Unlike natural language descriptions, specs have **precise structure** that AI can parse systematically:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│     SPEC     │────▶│    TESTS     │────▶│  IMPLEMENTATION   │
│              │     │              │     │                    │
│ • OpenAPI    │     │ • Generated  │     │ • AI-generated     │
│ • TypeScript │     │   from spec  │     │   to pass tests    │
│   interfaces │     │ • Reviewed   │     │ • Verified by       │
│ • JSON Schema│     │   by human   │     │   test suite        │
│ • Protobuf   │     │              │     │                    │
└──────────────┘     └──────────────┘     └──────────────────┘
       ▲                                          │
       └──────────────────────────────────────────┘
                 Spec evolves with learnings
```

**Why this works:** A spec defines the contract. Tests enforce the contract. The implementation fulfills the contract. AI excels at each transition.

---

## Spec Sources for Test Generation

| Spec Type | Test Coverage | AI Effectiveness |
|-----------|--------------|-----------------|
| **TypeScript interfaces** | Shape, types, required fields | ⭐⭐⭐⭐⭐ Excellent |
| **OpenAPI / Swagger** | Endpoints, params, responses | ⭐⭐⭐⭐⭐ Excellent |
| **JSON Schema** | Validation rules, constraints | ⭐⭐⭐⭐ Very good |
| **Protobuf definitions** | Message structure, field types | ⭐⭐⭐⭐ Very good |
| **Database schemas** | Column types, constraints, relations | ⭐⭐⭐ Good |
| **Requirements documents** | Behavior, acceptance criteria | ⭐⭐ Fair (ambiguity risk) |

---

## Pattern 1: Tests from TypeScript Interfaces

TypeScript interfaces are ideal test sources — they define exact shapes with type information.

### The Spec

```typescript
// types.ts — This is the specification
interface User {
  id: string;
  email: string;
  name: string;
  role: 'admin' | 'editor' | 'viewer';
  createdAt: Date;
  lastLogin?: Date;
}

interface CreateUserInput {
  email: string;
  name: string;
  role: 'admin' | 'editor' | 'viewer';
}

interface UserService {
  create(input: CreateUserInput): Promise<User>;
  findById(id: string): Promise<User | null>;
  updateRole(id: string, role: User['role']): Promise<User>;
  delete(id: string): Promise<void>;
}
```

### The Prompt

```
Given this TypeScript interface for a UserService, generate exhaustive tests
for an implementation. Cover:
1. Happy path for each method
2. Validation of required fields
3. Error cases (not found, invalid input, duplicates)
4. Return type shape verification
5. Edge cases for each parameter type

Use Jest and assume an in-memory implementation for testing.

[paste interfaces]
```

### AI-Generated Tests

```typescript
// user-service.test.ts — Generated from spec
import { UserService, CreateUserInput, User } from './types';
import { InMemoryUserService } from './user-service';

describe('UserService', () => {
  let service: UserService;

  beforeEach(() => {
    service = new InMemoryUserService();
  });

  describe('create', () => {
    const validInput: CreateUserInput = {
      email: 'user@example.com',
      name: 'Test User',
      role: 'viewer',
    };

    it('should create a user with all required fields', async () => {
      const user = await service.create(validInput);
      expect(user.email).toBe('user@example.com');
      expect(user.name).toBe('Test User');
      expect(user.role).toBe('viewer');
    });

    it('should generate a unique id', async () => {
      const user = await service.create(validInput);
      expect(user.id).toBeDefined();
      expect(typeof user.id).toBe('string');
      expect(user.id.length).toBeGreaterThan(0);
    });

    it('should set createdAt to current time', async () => {
      const before = new Date();
      const user = await service.create(validInput);
      const after = new Date();
      expect(user.createdAt.getTime()).toBeGreaterThanOrEqual(before.getTime());
      expect(user.createdAt.getTime()).toBeLessThanOrEqual(after.getTime());
    });

    it('should not set lastLogin on creation', async () => {
      const user = await service.create(validInput);
      expect(user.lastLogin).toBeUndefined();
    });

    it('should reject duplicate emails', async () => {
      await service.create(validInput);
      await expect(service.create(validInput)).rejects.toThrow('Email already exists');
    });

    it('should reject empty email', async () => {
      await expect(
        service.create({ ...validInput, email: '' })
      ).rejects.toThrow();
    });

    it('should reject invalid role', async () => {
      await expect(
        service.create({ ...validInput, role: 'superadmin' as any })
      ).rejects.toThrow();
    });

    it('should create users with each valid role', async () => {
      for (const role of ['admin', 'editor', 'viewer'] as const) {
        const user = await service.create({
          email: `${role}@example.com`,
          name: `${role} User`,
          role,
        });
        expect(user.role).toBe(role);
      }
    });
  });

  describe('findById', () => {
    it('should return the user when found', async () => {
      const created = await service.create({
        email: 'find@example.com',
        name: 'Find Me',
        role: 'editor',
      });
      const found = await service.findById(created.id);
      expect(found).toEqual(created);
    });

    it('should return null when user does not exist', async () => {
      const result = await service.findById('nonexistent-id');
      expect(result).toBeNull();
    });

    it('should return null for empty string id', async () => {
      const result = await service.findById('');
      expect(result).toBeNull();
    });
  });

  describe('updateRole', () => {
    it('should update the user role', async () => {
      const user = await service.create({
        email: 'role@example.com',
        name: 'Role User',
        role: 'viewer',
      });
      const updated = await service.updateRole(user.id, 'admin');
      expect(updated.role).toBe('admin');
    });

    it('should preserve other fields when updating role', async () => {
      const user = await service.create({
        email: 'preserve@example.com',
        name: 'Preserved',
        role: 'viewer',
      });
      const updated = await service.updateRole(user.id, 'editor');
      expect(updated.email).toBe('preserve@example.com');
      expect(updated.name).toBe('Preserved');
      expect(updated.id).toBe(user.id);
    });

    it('should throw when user does not exist', async () => {
      await expect(
        service.updateRole('nonexistent', 'admin')
      ).rejects.toThrow('User not found');
    });
  });

  describe('delete', () => {
    it('should remove the user', async () => {
      const user = await service.create({
        email: 'delete@example.com',
        name: 'Delete Me',
        role: 'viewer',
      });
      await service.delete(user.id);
      const result = await service.findById(user.id);
      expect(result).toBeNull();
    });

    it('should throw when user does not exist', async () => {
      await expect(service.delete('nonexistent')).rejects.toThrow('User not found');
    });
  });
});
```

---

## Pattern 2: Tests from OpenAPI Spec

```yaml
# openapi.yaml (excerpt)
paths:
  /api/products:
    get:
      summary: List products
      parameters:
        - name: category
          in: query
          schema:
            type: string
        - name: limit
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        '200':
          description: Product list
          content:
            application/json:
              schema:
                type: object
                properties:
                  items:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
                  total:
                    type: integer
```

### Prompt

```
Generate integration tests for this OpenAPI endpoint specification.
Test each parameter, validation rule, response shape, and error case.
Use supertest with an Express app.

[paste OpenAPI excerpt]
```

### Generated Tests

```typescript
// products-api.test.ts — Generated from OpenAPI spec
import request from 'supertest';
import { app } from './app';

describe('GET /api/products', () => {
  it('should return 200 with product list', async () => {
    const res = await request(app).get('/api/products');
    expect(res.status).toBe(200);
    expect(res.body).toHaveProperty('items');
    expect(res.body).toHaveProperty('total');
    expect(Array.isArray(res.body.items)).toBe(true);
    expect(typeof res.body.total).toBe('number');
  });

  it('should default limit to 20', async () => {
    const res = await request(app).get('/api/products');
    expect(res.body.items.length).toBeLessThanOrEqual(20);
  });

  it('should respect limit parameter', async () => {
    const res = await request(app).get('/api/products?limit=5');
    expect(res.body.items.length).toBeLessThanOrEqual(5);
  });

  it('should filter by category', async () => {
    const res = await request(app).get('/api/products?category=electronics');
    res.body.items.forEach((item: any) => {
      expect(item.category).toBe('electronics');
    });
  });

  it('should reject limit below minimum (1)', async () => {
    const res = await request(app).get('/api/products?limit=0');
    expect(res.status).toBe(400);
  });

  it('should reject limit above maximum (100)', async () => {
    const res = await request(app).get('/api/products?limit=101');
    expect(res.status).toBe(400);
  });

  it('should reject non-integer limit', async () => {
    const res = await request(app).get('/api/products?limit=abc');
    expect(res.status).toBe(400);
  });
});
```

---

## Pattern 3: Tests from Requirements Document

Requirements are less structured, but AI can still extract testable behaviors:

### The Requirement

```markdown
## User Registration

- Users must provide email, password, and display name
- Email must be unique across the system
- Password must be at least 8 characters with one uppercase, one number
- Display name must be 2-50 characters
- On success, send a verification email and return the user profile
- On duplicate email, return a 409 Conflict error
```

### Prompt

```
Convert these requirements into a test suite. Each requirement
should map to one or more test cases. Include the requirement
text as a comment above each test group.

[paste requirements]
```

### Generated Tests

```typescript
describe('User Registration', () => {
  // "Users must provide email, password, and display name"
  it('should reject registration without email', async () => {
    await expect(register({ password: 'Pass1234', name: 'Test' }))
      .rejects.toThrow('Email is required');
  });

  it('should reject registration without password', async () => {
    await expect(register({ email: 'a@b.com', name: 'Test' }))
      .rejects.toThrow('Password is required');
  });

  it('should reject registration without display name', async () => {
    await expect(register({ email: 'a@b.com', password: 'Pass1234' }))
      .rejects.toThrow('Display name is required');
  });

  // "Password must be at least 8 characters with one uppercase, one number"
  it('should reject short passwords', async () => {
    await expect(register({ email: 'a@b.com', password: 'Pass1', name: 'T' }))
      .rejects.toThrow();
  });

  it('should reject passwords without uppercase', async () => {
    await expect(register({ email: 'a@b.com', password: 'password1', name: 'Test' }))
      .rejects.toThrow();
  });

  it('should reject passwords without numbers', async () => {
    await expect(register({ email: 'a@b.com', password: 'Password', name: 'Test' }))
      .rejects.toThrow();
  });

  // "Display name must be 2-50 characters"
  it('should reject display name shorter than 2 characters', async () => {
    await expect(register({ email: 'a@b.com', password: 'Pass1234', name: 'A' }))
      .rejects.toThrow();
  });

  it('should reject display name longer than 50 characters', async () => {
    await expect(register({ email: 'a@b.com', password: 'Pass1234', name: 'A'.repeat(51) }))
      .rejects.toThrow();
  });

  // "On duplicate email, return a 409 Conflict error"
  it('should return 409 for duplicate email', async () => {
    await register({ email: 'dup@test.com', password: 'Pass1234', name: 'First' });
    await expect(register({ email: 'dup@test.com', password: 'Pass1234', name: 'Second' }))
      .rejects.toMatchObject({ status: 409 });
  });
});
```

---

## Best Practices

- **Start with the most formal spec available** — TypeScript interfaces > OpenAPI > prose requirements
- **Review generated tests for completeness** — AI may miss implicit requirements
- **Add tests for behaviors the spec doesn't cover** — specs often omit error handling details
- **Keep traceability** — comment which spec section each test group covers
- **Update tests when the spec changes** — re-run the generation prompt against the new spec
