# The Constitution

> Defining invariant project principles that govern all specifications.

---

## What Is a Constitution?

A **constitution** is a project-level document that defines principles, constraints, and standards that apply to *every* feature specification. It's the "law" that all specs must follow.

```
Constitution = Project-wide invariants
Specification = Feature-specific requirements
```

---

## Why You Need One

Without a constitution, every spec reinvents the wheel:

```
❌  Without Constitution:
  Spec A: "Use React for this feature"
  Spec B: "This feature should work on mobile"
  Spec C: "Follow accessibility standards"
  → Every spec repeats the same constraints

✅  With Constitution:
  Constitution: "All features use React 19, must be responsive, must meet WCAG 2.1 AA"
  Spec A: [focuses only on feature-specific requirements]
  Spec B: [focuses only on feature-specific requirements]
```

---

## Constitution Template

```markdown
# Project Constitution

## Identity
- **Project**: [Name]
- **Type**: [Web app / API / CLI / Library]
- **Team**: [Size, roles]

## Design Principles
1. [Principle 1]: [Why it matters]
2. [Principle 2]: [Why it matters]
3. [Principle 3]: [Why it matters]

## Technology Stack
| Layer | Technology | Version |
|-------|-----------|---------|
| Frontend | React + TypeScript | 19 / 5.6 |
| Backend | Node.js + Fastify | 22 / 5.x |
| Database | PostgreSQL + Prisma | 16 / 6.x |
| Testing | Vitest + Playwright | Latest |

## Coding Standards
- TypeScript strict mode, no `any` types
- ESLint + Prettier configuration in repo
- All functions documented with JSDoc
- Maximum file length: 300 lines

## Quality Requirements
- Test coverage: >80% for new code
- No critical/high security vulnerabilities
- Lighthouse performance score: >90
- WCAG 2.1 AA accessibility compliance

## API Conventions
- RESTful design with JSON:API format
- Consistent error response format
- Rate limiting on all public endpoints
- OpenAPI documentation required

## Security Constraints
- All user input sanitized
- Authentication via JWT with refresh tokens
- PII encrypted at rest
- OWASP Top 10 mitigations required
```

---

## Constitution vs Copilot Instructions

| Aspect | Constitution | copilot-instructions.md |
|--------|-------------|------------------------|
| **Scope** | Project architecture and standards | AI behavior and coding style |
| **Audience** | Humans + AI agents | AI agents primarily |
| **Consumed by** | SDD workflow tools | GitHub Copilot |
| **Contains** | Business rules, tech stack, quality standards | Code patterns, naming conventions |
| **Overlap** | Some — both define standards | Some — both define standards |

### Integration Strategy
```
constitution.md:  "All APIs follow RESTful design with OpenAPI docs"
                        ↓ informs ↓
copilot-instructions.md:  "When generating API code, use Fastify with
                           route schemas. Always include OpenAPI annotations."
```

---

## How AI Tools Use the Constitution

### Spec Kit
```bash
speckit constitution    # Create/edit constitution
# Stored in .specify/constitution.md
# Referenced automatically by /specify, /plan, /tasks
```

### AWS Kiro
Constitution equivalent: **Steering files** in `.kiro/steering/`

### Manual Integration
When using any AI tool, include constitution context:
```
"Follow our project constitution:
 - TypeScript strict mode
 - Prisma ORM for all database access
 - All endpoints require authentication
 [paste relevant sections]"
```

---

## Maintaining the Constitution

### When to Update
- New technology added to stack
- New quality requirement identified
- New compliance requirement discovered
- Team agrees on new convention

### How to Update
1. Propose change as a PR with rationale
2. Team reviews (like any code change)
3. Update all affected specs if necessary
4. Communicate change to team

---

*Next: [Quality Gates →](quality-gates.md)*
