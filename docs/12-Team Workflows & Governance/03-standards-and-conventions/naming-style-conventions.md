# Naming and Style Conventions for AI

> Keeping AI output consistent with your team's style. Without explicit conventions in your instruction files, AI generates code in its "default style" — which rarely matches your team's preferences.

---

## The Problem

AI tools have a "default style" based on their training data. Without specific instructions:

```typescript
// AI's default style (generic)
function getUserById(id: string): Promise<User | null> {
  const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
  if (!user) return null;
  return user;
}

// Your team's style (specific)
async function findUserById(userId: UserId): Promise<Result<User, AppError>> {
  const result = await userRepository.findById(userId);
  if (result.isNone()) {
    return err(new AppError('USER_NOT_FOUND', `User ${userId} not found`));
  }
  return ok(result.unwrap());
}
```

Both are valid code. But if your codebase follows the second pattern and AI generates the first, you have inconsistency.

---

## What to Specify in Instruction Files

### Naming Conventions

Be explicit about every naming pattern:

```markdown
## Naming Conventions

### Variables and Functions
- Use camelCase: `userId`, `getActiveUsers`
- Boolean variables: prefix with is/has/should: `isActive`, `hasPermission`
- Event handlers: prefix with handle: `handleSubmit`, `handleClick`
- Async functions: no special prefix, use async/await

### Types and Interfaces
- Use PascalCase: `UserProfile`, `ApiResponse`
- Interface prefix: none (no I prefix): `UserService` not `IUserService`
- Generic types: single letter or descriptive: `T`, `TResult`, `TError`
- Enum values: PascalCase: `UserRole.Admin`, `Status.Active`

### Constants
- Module-level: UPPER_SNAKE_CASE: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`
- Object constants: camelCase keys: `const config = { maxRetries: 3 }`

### Files and Directories
- Source files: kebab-case: `user-service.ts`, `api-client.ts`
- Test files: same name + .test: `user-service.test.ts`
- React components: PascalCase: `UserProfile.tsx`
- Directories: kebab-case: `user-management/`, `api-routes/`
```

### File Organization

Specify where things go:

```markdown
## File Organization

### Import Order
1. Node.js built-in modules
2. External dependencies (npm packages)
3. Internal absolute imports (@/...)
4. Relative imports
5. Type-only imports

Separate each group with a blank line.

### File Structure
Every source file follows this order:
1. Imports
2. Type definitions
3. Constants
4. Helper functions (private)
5. Main export (class/function/component)

### Module Exports
- Use named exports, not default exports
- Export types separately: `export type { UserProps }`
- Re-export from index files for public API
```

### Comment Style

Tell AI when and how to comment:

```markdown
## Comments

### When to Comment
- Complex business logic that isn't obvious from code
- Workarounds with links to issues/tickets
- Public API functions (JSDoc)

### When NOT to Comment
- Obvious operations (// increment counter)
- Restating the code in English
- Commented-out code (delete it, use git history)

### Format
- JSDoc for public functions:
  /**
   * Finds active users who haven't logged in for the given number of days.
   * @param inactiveDays - Number of days since last login
   * @returns Active users matching the inactivity criteria
   */

- Inline comments: above the line, not at end of line
- TODO format: // TODO(username): description (#issue-number)
```

### Error Handling Patterns

```markdown
## Error Handling

### Pattern: Result Type
Use Result<T, AppError> for all operations that can fail:
```typescript
// ✅ Correct
async function findUser(id: UserId): Promise<Result<User, AppError>> {
  try {
    const user = await userRepo.findById(id);
    if (!user) return err(AppError.notFound('User', id));
    return ok(user);
  } catch (e) {
    return err(AppError.internal('Failed to fetch user', e));
  }
}

// ❌ Incorrect — do not throw
async function findUser(id: UserId): Promise<User> {
  const user = await userRepo.findById(id);
  if (!user) throw new Error('User not found');
  return user;
}
```

### Error Messages
- User-facing: clear, actionable, no internal details
- Developer-facing (logs): include context, stack trace, request ID
- Never expose: database names, file paths, stack traces to API consumers
```

### Logging Format

```markdown
## Logging

### Logger
Use the structured logger from src/utils/logger.ts

### Log Levels
- error: Unrecoverable failures, data loss, security events
- warn: Recoverable issues, deprecated usage, retry attempts
- info: Business events, request/response summary
- debug: Detailed technical information (disabled in production)

### Format
```typescript
// ✅ Structured logging
logger.info('User login successful', {
  userId: user.id,
  method: 'oauth',
  duration: elapsed,
});

// ❌ String concatenation
console.log('User ' + user.id + ' logged in via ' + method);
```
```

---

## Path-Specific Conventions

Different parts of your codebase may have different conventions. Use instruction files or directory-specific guidance:

### Frontend (React) Conventions
```markdown
## Frontend Conventions (src/components/)

### Component Structure
- Functional components only (no class components)
- Props interface defined above component
- Hooks at the top of the component
- Event handlers defined inside the component
- Styled with CSS modules or Tailwind (no inline styles)

### State Management
- Local state: useState for component-specific state
- Shared state: Zustand store (see src/stores/)
- Server state: TanStack Query (see src/hooks/queries/)

### Naming
- Components: PascalCase files and exports
- Hooks: use prefix: useAuth, useUserProfile
- Event handlers: handle prefix: handleSubmit
- CSS modules: styles.container, styles.header
```

### Backend (API) Conventions
```markdown
## Backend Conventions (src/routes/, src/services/)

### Route Handlers
- Express Router with typed request/response
- Validation middleware using zod schemas
- Error responses follow RFC 7807 Problem Details
- All routes require authentication unless explicitly public

### Service Layer
- Business logic lives in services, not route handlers
- Services return Result<T, AppError>
- Services are stateless (no instance variables)
- Dependencies injected via constructor

### Database Access
- Repository pattern (see src/repos/)
- All queries use parameterized statements
- Transactions for multi-table operations
- No raw SQL outside repository files
```

---

## Before/After: AI Output Without vs. With Style Instructions

### Without Style Instructions

```typescript
// AI generates in its "default" style
const getUsers = async (req, res) => {
  try {
    let users = await db.query("SELECT * FROM users WHERE active = true");
    users = users.filter(u => u.role !== "deleted");
    res.json({ success: true, data: users });
  } catch (err) {
    console.error(err);
    res.status(500).json({ success: false, error: "Something went wrong" });
  }
};

export default getUsers;
```

**Issues:**
- ❌ No TypeScript types
- ❌ Raw SQL instead of repository pattern
- ❌ `console.error` instead of structured logger
- ❌ Inconsistent response format
- ❌ Default export instead of named export
- ❌ Filtering in application code instead of query
- ❌ Generic error message

### With Style Instructions

```typescript
// AI generates following team conventions
import { Router, Request, Response } from 'express';
import { z } from 'zod';

import { userRepository } from '@/repos/user-repository';
import { AppError } from '@/types/errors';
import { logger } from '@/utils/logger';

const listUsersQuerySchema = z.object({
  role: z.enum(['admin', 'member', 'viewer']).optional(),
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().positive().max(100).default(20),
});

/**
 * Lists active users with optional role filter and pagination.
 */
export async function handleListUsers(
  req: Request,
  res: Response,
): Promise<void> {
  const query = listUsersQuerySchema.safeParse(req.query);

  if (!query.success) {
    res.status(400).json(
      AppError.validation('Invalid query parameters', query.error.issues),
    );
    return;
  }

  const result = await userRepository.findActive(query.data);

  if (result.isErr()) {
    logger.error('Failed to list users', { error: result.error });
    res.status(500).json(AppError.internal('Unable to retrieve users'));
    return;
  }

  const { users, total } = result.value;

  res.json({
    data: users,
    pagination: {
      page: query.data.page,
      limit: query.data.limit,
      total,
    },
  });
}
```

**Improvements:**
- ✅ Full TypeScript types
- ✅ Repository pattern for data access
- ✅ Structured logger
- ✅ Consistent response format with pagination
- ✅ Named export
- ✅ Zod validation on input
- ✅ Proper error handling with AppError

---

## Testing Your Conventions

After updating your instruction file with style conventions, verify:

```
1. Generate a new API endpoint → Check naming, patterns, error handling
2. Generate a React component → Check structure, hooks, styling approach
3. Generate a test file → Check naming, setup, assertion style
4. Generate a utility function → Check JSDoc, error handling, types
5. Ask AI to refactor existing code → Check if it preserves your conventions
```

If the output doesn't match, refine the instruction file wording until it does.
