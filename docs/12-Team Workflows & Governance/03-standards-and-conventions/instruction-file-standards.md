# Instruction File Standards

> The instruction file is your team's agreement on how AI should generate code. It lives in version control, goes through PR review, and is the single most impactful artifact in team AI adoption.

---

## The Instruction File as Team Agreement

An instruction file isn't just a config file — it's a **social contract** between team members, encoded in a format that AI tools can read. Everyone contributes, everyone follows, and changes go through review.

```
Team Convention → Instruction File → AI Tool → Consistent Output
                     ↑                              ↓
                 PR Review ← Feedback ← Code Review Results
```

---

## copilot-instructions.md Standard

The primary instruction file for GitHub Copilot. Place in `.github/copilot-instructions.md` or the repo root.

### Template

```markdown
# Project: [Project Name]

## Overview
[1-2 sentence project description for AI context]

## Tech Stack
- Language: [e.g., TypeScript 5.x, strict mode]
- Framework: [e.g., Next.js 14, App Router]
- Database: [e.g., PostgreSQL via Prisma ORM]
- Testing: [e.g., Vitest + Testing Library]
- CI/CD: [e.g., GitHub Actions]

## Code Conventions

### Naming
- Variables and functions: camelCase
- Types and interfaces: PascalCase
- Constants: UPPER_SNAKE_CASE
- Files: kebab-case.ts
- Test files: *.test.ts (co-located with source)

### Patterns
- Error handling: Use Result<T, AppError> pattern, never throw
- Validation: Zod schemas for all external input
- API responses: Follow RFC 7807 problem details for errors
- State management: [team's approach]
- Logging: Structured JSON via [logger library]

### File Organization
- src/services/     — Business logic
- src/routes/       — API route handlers
- src/models/       — Data models and schemas
- src/utils/        — Shared utilities
- src/types/        — TypeScript type definitions
- tests/            — Integration and E2E tests

## Testing Requirements
- All new functions must have unit tests
- API endpoints require integration tests
- Minimum 80% coverage on new code
- Test command: `npm test`
- Coverage command: `npm run test:coverage`

## Security Rules
- NEVER use eval() or Function() constructor
- All SQL must use parameterized queries
- User input must be validated before processing
- No hardcoded secrets or API keys
- Sanitize all output to prevent XSS

## Forbidden Patterns
- No `any` types in TypeScript
- No `console.log` in production code (use logger)
- No synchronous file I/O in request handlers
- No wildcard imports (import * as)
- No mutation of function parameters

## Build and Run
- Install: `npm install`
- Dev: `npm run dev`
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
```

---

## AGENTS.md for Cross-Tool Compatibility

`AGENTS.md` serves as a universal instruction file readable by multiple AI tools. Place in the repo root.

### Template

```markdown
# AGENTS.md — Cross-Tool AI Instructions

This file provides instructions for any AI coding tool working
in this repository.

## Project Context
[Same as copilot-instructions.md overview section]

## Universal Rules
1. Follow existing code patterns — look at neighboring files before generating
2. All code must be type-safe — no implicit any, no type assertions unless documented
3. Tests required for all new code — no exceptions
4. Security-first — validate inputs, parameterize queries, sanitize outputs
5. No secrets in code — use environment variables

## Code Generation Guidelines
- Match the style of surrounding code
- Include error handling for all external calls
- Add JSDoc comments for public functions
- Generate tests alongside implementation

## Review Instructions
When reviewing code, focus on:
- Security vulnerabilities (injection, auth bypass, data exposure)
- Error handling completeness
- Edge cases (null, empty, boundary values)
- Performance implications
- Adherence to project patterns
```

---

## CLAUDE.md for Claude Code Users

Claude Code reads `CLAUDE.md` from the repo root. Structure it similarly but include Claude-specific features.

### Template

```markdown
# CLAUDE.md

## Project
[Project name and description]

## Conventions
[Same conventions as copilot-instructions.md]

## Workflow Preferences
- Always run tests after making changes: `npm test`
- Commit frequently with descriptive messages
- Ask before modifying files outside the current task scope

## Tool Permissions
- Allowed: file read/write, terminal commands, git operations
- Ask first: dependency installation, config file changes
- Never: production deployments, secret access, destructive operations
```

---

## Version Control and Governance

### File Locations
```
repo-root/
├── .github/
│   └── copilot-instructions.md    # GitHub Copilot instructions
├── AGENTS.md                       # Cross-tool instructions
├── CLAUDE.md                       # Claude Code instructions
└── .copilotignore                  # Files AI should not read
```

### Change Process

All instruction file changes must go through PR review:

1. **Propose** — Create a branch with the proposed change
2. **Explain** — PR description explains why the change is needed
3. **Review** — Minimum 2 approvals required
4. **Test** — Verify AI output changes as expected with the new rules
5. **Merge** — Squash merge with clear commit message

### Who Can Modify Instruction Files

| Role | Permissions |
|---|---|
| Team lead | Full edit, final approval |
| Senior developers | Propose and approve changes |
| All developers | Propose changes via PR |
| New developers | Suggest changes (buddy submits PR) |

### Changelog

Maintain a changelog within the instruction file or as a companion file:

```markdown
## Changelog
- 2025-07-15: Added forbidden pattern for wildcard imports
- 2025-06-01: Updated test coverage requirement from 70% to 80%
- 2025-04-15: Initial version — team kickoff
```

---

## Quarterly Review Cycle

Every quarter, the team reviews and updates instruction files:

### Review Agenda (30-60 minutes)
1. **What's working?** — Rules that improved output quality
2. **What's not working?** — Rules that are ignored or produce worse output
3. **What's missing?** — Common issues that could be prevented by new rules
4. **What's outdated?** — Rules that no longer apply
5. **Feedback from new members** — Fresh perspective on clarity

### Review Checklist
- [ ] All rules are still relevant to the current tech stack
- [ ] Build/test/lint commands are accurate
- [ ] File organization section matches actual structure
- [ ] Security rules cover recent vulnerability patterns
- [ ] Forbidden patterns list is up to date
- [ ] New team members can understand the file without additional context

---

## Tips for Effective Instruction Files

✅ **Be specific and concrete**
```markdown
✅ "Use camelCase for variables and functions"
❌ "Follow good naming conventions"
```

✅ **Include examples for complex rules**
```markdown
✅ "Error handling: return Result<T, AppError> instead of throwing
   Example: return err(new AppError('NOT_FOUND', 'User not found'))"
❌ "Handle errors properly"
```

✅ **Reference existing code**
```markdown
✅ "Follow the repository pattern in src/repos/UserRepo.ts"
❌ "Use the repository pattern"
```

✅ **Keep it focused**
```markdown
✅ One instruction file per repository, updated quarterly
❌ Multiple instruction files that contradict each other
```

✅ **Test your rules**
```markdown
After adding a new rule, test it:
1. Start a fresh AI conversation
2. Prompt something that should trigger the rule
3. Verify the AI follows it
4. If not, refine the rule's wording
```

## Next Steps

- [Build a shared prompt library →](prompt-library.md)
- [Set code quality standards for AI output →](code-quality-standards.md)
- [Define naming and style conventions →](naming-style-conventions.md)
- [Design your AI code review workflow →](../04-review-processes/ai-code-review-workflow.md)
