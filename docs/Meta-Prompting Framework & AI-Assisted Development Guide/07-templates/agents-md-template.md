# AGENTS.md Template

> Copy this to the root of your repository as `AGENTS.md` and customize.

---

```markdown
# AGENTS.md

## Project
<!-- Brief description for any AI agent encountering this repo -->
[Project name]: [One-sentence description].
Stack: [Language], [Framework], [Database], [Key libraries].

## Setup
- Install: `npm install`
- Dev: `npm run dev`
- Build: `npm run build`
- Test: `npm run test`
- Lint: `npm run lint`

## Code Style
- [Language] [strict mode / version requirement]
- [Component/module pattern]
- [Naming conventions]
- [Import/export conventions]
- [Styling approach]

## Architecture
<!-- Brief description of major architectural elements -->
- [Framework pattern] (e.g., "Next.js App Router with RSC")
- [Data access pattern] (e.g., "Repository pattern in src/repositories/")
- [API pattern] (e.g., "REST APIs in src/app/api/")
- [State management] (e.g., "React Server Components + client hooks")
- [Database] (e.g., "PostgreSQL via Prisma ORM")

## Do
- Run tests before committing
- Follow existing patterns in the codebase
- Use the project logger for all logging
- Write tests for all new functionality
- Validate user input on the server side

## Don't
- Don't commit secrets or credentials
- Don't add dependencies without team approval
- Don't modify files in `vendor/` or `generated/`
- Don't disable linting or type checking rules
- Don't use `console.log` in production code
- Don't bypass input validation

## Testing
- Framework: [Jest / Vitest / etc.]
- Run single test: `npm run test -- path/to/file.test.ts`
- All PRs require passing tests
- Test patterns: [describe/it nesting, behavior-driven, etc.]

## Safety
- Never commit `.env` files or API keys
- Validate and sanitize all user input
- Use parameterized queries for database access
- Follow the principle of least privilege for API permissions
```
