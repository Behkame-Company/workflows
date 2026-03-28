# AI Code Review Workflow

> A modified review process for AI-generated code. Traditional review focuses on "what the developer intended." AI review focuses on "what was actually generated" — because AI code is often syntactically perfect but logically subtle.

---

## Why AI Code Review Is Different

| Traditional Code Review | AI Code Review |
|---|---|
| Developer wrote every line intentionally | Developer accepted/modified generated code |
| Style issues are common | Style is usually perfect |
| Logic reflects developer's reasoning | Logic may reflect AI's assumptions |
| Developer can explain every decision | Developer may not have reviewed every detail |
| Focus: style + logic | Focus: correctness + edge cases + security |

The core difference: **AI-generated code often looks more polished but needs deeper logical scrutiny.**

---

## The Five-Step Workflow

```
┌─────────────┐    ┌────────────────┐    ┌──────────────────┐
│  1. AI      │───▶│  2. Automated  │───▶│  3. AI-Assisted  │
│  Generates  │    │  Checks Run    │    │  Pre-Review      │
│  PR         │    │  (CI/CD)       │    │  (Optional)      │
└─────────────┘    └────────────────┘    └──────────────────┘
                                                  │
                   ┌────────────────┐    ┌────────▼─────────┐
                   │  5. Merge      │◀───│  4. Human Deep   │
                   │  (if approved) │    │  Review          │
                   └────────────────┘    └──────────────────┘
```

### Step 1: AI Generates PR

The developer creates a PR that includes AI-generated code. The PR must:

- Use the team's [PR template](pr-templates.md) with AI disclosure
- Clearly indicate which parts are AI-generated
- Describe what the AI was asked to do (prompt or intent)
- Note areas that need special attention

```markdown
## AI Disclosure
- **Tool:** GitHub Copilot CLI
- **Scope:** Full implementation of UserService class
- **Prompt summary:** "Create UserService with CRUD operations following 
  our repository pattern"
- **Human modifications:** Added custom validation in updateUser, 
  fixed error handling in deleteUser
- **Areas needing attention:** Concurrent access handling in updateUser
```

### Step 2: Automated Checks Run

CI/CD pipeline runs the same checks for all code:

```yaml
# Automated quality gates
checks:
  - lint          # ESLint, Prettier
  - type-check    # TypeScript strict mode
  - test          # Unit and integration tests
  - coverage      # Minimum coverage threshold
  - sast          # Static analysis security testing
  - secrets       # Secret scanning
  - deps          # Dependency audit
```

**Gate rules:**
- ✅ All checks must pass before human review begins
- ✅ Failing checks are addressed by the developer, not the reviewer
- ❌ Do not skip checks for AI-generated code
- ❌ Do not suppress warnings without documented justification

### Step 3: AI-Assisted Pre-Review (Optional)

Use a separate AI tool to review the AI-generated code. This catches a category of issues that automated checks miss:

```
Prompt for AI pre-review:
"Review this code change for:
1. Logic errors or incorrect assumptions
2. Missing error handling
3. Security vulnerabilities not caught by SAST
4. Edge cases that aren't tested
5. Deviations from project patterns

Focus on correctness, not style."
```

**What AI pre-review catches:**
- Hallucinated imports or API calls
- Logic that handles happy path but fails on edge cases
- Pattern deviations that linters don't check
- Missing null checks or type guards

**What AI pre-review misses:**
- Business logic correctness
- Architectural appropriateness
- Performance implications in context
- UX impact

### Step 4: Human Deep Review

The human reviewer focuses on what automated tools and AI pre-review can't catch:

#### The 80/20 Rule

Focus 80% of your review time on these areas:

**Boundaries and Trust Transitions**
```
Where does data cross a trust boundary?
- User input → application: validation complete?
- Application → database: queries parameterized?
- External API → application: errors handled?
- Application → user: sensitive data filtered?
```

**Error Paths**
```
What happens when things go wrong?
- Network timeout: retry? fail gracefully? hang forever?
- Invalid data: clear error? silent corruption? crash?
- Permission denied: appropriate response? info leak?
- Resource exhausted: graceful degradation? cascade failure?
```

**Logic Correctness**
```
Does this code do what it's supposed to?
- Business rules accurately implemented?
- Edge cases handled (null, empty, boundary)?
- State transitions correct (especially in async code)?
- Race conditions possible?
```

#### Review Depth by AI Scope

| AI Scope | Review Depth | Focus |
|---|---|---|
| Autocomplete (single line) | Light | Glance for correctness |
| Function generation | Standard | Logic, edge cases, patterns |
| Full feature generation | Deep | Architecture, security, integration |
| Refactoring | Standard | Behavior preservation, test coverage |
| Test generation | Standard | Test quality, meaningful assertions |

### Step 5: Merge

After human approval:

- Squash merge or merge per team convention
- Ensure CI passes on final commit
- AI disclosure is preserved in PR history

---

## Workflow Diagram

```
Developer creates PR with AI-generated code
          │
          ▼
    ┌──────────┐
    │ PR has    │──No──▶ Request AI disclosure
    │ AI       │         from developer
    │ disclosure│
    └────┬─────┘
         │ Yes
         ▼
    ┌──────────┐
    │ Automated │──Fail──▶ Developer fixes
    │ checks    │          and re-pushes
    │ pass?     │
    └────┬─────┘
         │ Pass
         ▼
    ┌──────────┐
    │ AI pre-  │──Issues──▶ Developer addresses
    │ review   │            findings
    │ (optional)│
    └────┬─────┘
         │ Clean
         ▼
    ┌──────────┐
    │ Human    │──Changes──▶ Developer revises
    │ review   │  requested
    └────┬─────┘
         │ Approved
         ▼
    ┌──────────┐
    │ Merge    │
    └──────────┘
```

---

## Common Review Scenarios

### Scenario: AI-Generated API Endpoint

**What to check:**
- [ ] Authentication required and implemented correctly
- [ ] Input validation covers all fields with appropriate rules
- [ ] Database queries are parameterized
- [ ] Error responses don't leak internal details
- [ ] Rate limiting considered
- [ ] Pagination for list endpoints
- [ ] Appropriate HTTP status codes

### Scenario: AI-Generated React Component

**What to check:**
- [ ] Component handles loading, error, and empty states
- [ ] Accessibility attributes present (aria labels, roles)
- [ ] Event handlers are properly typed
- [ ] No unnecessary re-renders (memoization where needed)
- [ ] Follows team component patterns

### Scenario: AI-Generated Test Suite

**What to check:**
- [ ] Tests verify behavior, not implementation details
- [ ] Edge cases are covered (null, empty, boundary)
- [ ] Tests are independent (no shared state between tests)
- [ ] Test descriptions accurately describe what's being tested
- [ ] No tautological tests (asserting what the code does, not what it should do)

---

## Review Time Guidelines

| PR Size | AI Scope | Suggested Review Time |
|---|---|---|
| Small (< 50 lines) | Autocomplete | 5-10 minutes |
| Medium (50-200 lines) | Function/module | 15-30 minutes |
| Large (200-500 lines) | Full feature | 30-60 minutes |
| Extra large (500+ lines) | Major feature | Split the PR |

> **Tip:** AI-generated PRs larger than 500 lines should be split into smaller, reviewable chunks.

## Next Steps

- [Understand the human-AI review balance →](human-ai-review-balance.md)
- [Use PR templates for AI-assisted work →](pr-templates.md)
- [Apply review checklists →](review-checklists.md)
- [Set code quality standards →](../03-standards-and-conventions/code-quality-standards.md)
