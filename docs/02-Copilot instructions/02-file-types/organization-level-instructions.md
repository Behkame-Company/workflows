# Organization-Level Instructions

> Policies and standards that apply across all repositories.

---

## Overview

| Property | Value |
|----------|-------|
| **Location** | GitHub.com → Organization Settings → Copilot → Custom Instructions |
| **Scope** | All repositories in the organization |
| **Auto-loaded** | Yes — included in every Copilot interaction |
| **Priority** | Lowest — repository and personal override |
| **Access** | Organization owners and Copilot admins |

---

## Setting Up Organization Instructions

### Via GitHub.com

1. Go to your organization on GitHub.com
2. **Settings** → **Copilot** → **Policies** → **Custom Instructions**
3. Enter your instructions in the text field
4. Save

### What Gets Sent

Organization instructions are included in the system context for every Copilot interaction across all repos in the org. Users cannot see the full instructions but will see their effects.

---

## What Belongs at Organization Level

Organization instructions should contain **universally applicable** rules that are true for **every** repository. If a rule has exceptions, it belongs at the repository level.

### ✅ Good: Universal Policies

```markdown
## Language & Framework Standards
- All new services must use TypeScript with strict mode
- All API endpoints must validate input

## Security
- Never commit secrets, API keys, or credentials to source control
- Use parameterized queries; never concatenate SQL strings
- Sanitize all user input before rendering in HTML

## Documentation
- All public functions must have JSDoc documentation
- All API endpoints must have OpenAPI/Swagger annotations
- README.md must include setup and deployment instructions

## Code Quality
- All code changes must include appropriate tests
- Follow the organization ESLint configuration
- Use conventional commit format for all commit messages
```

### ❌ Bad: Too Specific for Org Level

```markdown
# Don't put these at org level:
- Use React 18 with hooks         # Not all repos use React
- Use Prisma for database access   # Not all repos use Prisma
- Use Tailwind for styling          # Not all repos use Tailwind
- Run `npm test` for testing        # Not all repos use npm
```

---

## Sizing and Limits

Keep organization instructions **short** — 5-15 lines maximum.

**Why?**
1. These are loaded for **every** interaction across **every** repo
2. They consume context budget in **every** request
3. Longer org instructions leave less room for repo-specific context
4. Most useful rules are repo-specific, not org-wide

```
Bad:  50-line organization policy document
Good: 10-line essential rules that apply universally
```

---

## Layering Strategy

### Principle: General → Specific

```
Organization:  "Validate all input"
Repository:    "Validate input with Zod using schemas in src/validators/"
Path-specific: "API routes must use the UserInput schema from validators/user.ts"
```

Each layer adds specificity without contradiction.

### Principle: Org Sets Direction, Repo Sets Implementation

```
Organization:  "Write tests for all code changes"
Repository:    "Use Vitest for unit tests and Playwright for E2E tests"
Path-specific: "Test React components with @testing-library/react"
```

---

## Governance Model

### Who Writes Organization Instructions

| Role | Responsibility |
|------|---------------|
| **CTO / VP Engineering** | Approve org-wide policies |
| **Platform Team** | Draft and maintain instructions |
| **Security Team** | Review security-related rules |
| **Developer Experience** | Test that rules are effective |

### Review Cadence

- **Quarterly**: Full review of org instructions
- **Monthly**: Check for stale or ineffective rules
- **On-demand**: When new technology decisions are made

### Change Process

1. Proposed change → PR or RFC discussion
2. Test in a pilot repository first
3. Review impact on existing repo-level instructions
4. Roll out to organization settings
5. Communicate change to all developers

---

## Interaction with Repository Instructions

### No Conflict (Complementary)

```
Org:   "Write documentation for public APIs"
Repo:  "Use JSDoc format for documentation"
```
→ Both apply together. Copilot writes JSDoc documentation for public APIs. ✅

### Implicit Override

```
Org:   "Use CSS Modules for styling"
Repo:  "Use Tailwind CSS for all styling"
```
→ Ambiguous. Copilot may mix approaches. ❌

**Better approach** — don't include styling preferences at org level. Let each repo decide.

### Explicit Override

```
Repo copilot-instructions.md:
"Note: This project uses Tailwind CSS, not the org-standard CSS Modules."
```
→ Clear and explicit. Copilot understands the override. ✅

---

## Real-World Organization Template

```markdown
## Standards
- TypeScript strict mode for all new code
- Conventional Commits for all commit messages (feat:, fix:, docs:, etc.)

## Security
- Never commit secrets, credentials, or API keys
- Use parameterized queries for all database operations
- Sanitize user-facing input to prevent XSS

## Quality
- All PRs must include tests for new/changed behavior
- All public APIs must have documentation (JSDoc or equivalent)
- Follow the organization's linter configuration

## Process
- Reference the GitHub issue number in commit messages
- PRs require at least one review before merge
```

**Total: 12 rules, ~150 words** — compact and universally applicable.
