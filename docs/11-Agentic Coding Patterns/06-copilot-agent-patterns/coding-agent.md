# Copilot Coding Agent

> GitHub's cloud-based autonomous coding agent that turns issues into pull requests — assign an issue, get a working PR back.

---

## How It Works

The Copilot Coding Agent operates as a fully autonomous cloud developer. When you assign it to an issue, it spins up a secure cloud container, creates a branch, implements the solution, and opens a pull request — all without touching your local machine.

```
┌─────────────────────────────────────────────────────────────────┐
│                  Copilot Coding Agent Workflow                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────┐    ┌──────────┐    ┌───────────────┐            │
│   │  GitHub   │    │  Agent   │    │    Cloud      │            │
│   │  Issue    │───▶│  Picks   │───▶│   Container   │            │
│   │  Assigned │    │  Up      │    │   Spins Up    │            │
│   └──────────┘    └──────────┘    └───────┬───────┘            │
│                                           │                     │
│                                           ▼                     │
│   ┌──────────┐    ┌──────────┐    ┌───────────────┐            │
│   │   Pull   │    │  Tests   │    │  Implements   │            │
│   │  Request │◀───│  Pass?   │◀───│   Solution    │            │
│   │  Opened  │    │  Iterate │    │   on Branch   │            │
│   └──────────┘    └──────────┘    └───────────────┘            │
│                                                                 │
│   Developer reviews PR ──▶ Merge or request changes             │
│                            (agent iterates on feedback)         │
└─────────────────────────────────────────────────────────────────┘
```

### Step-by-Step Flow

1. **Issue Creation** — Write a clear, well-scoped GitHub issue
2. **Assignment** — Assign the issue to `@copilot` (or use the Copilot button)
3. **Branch Creation** — Agent creates a feature branch automatically
4. **Environment Setup** — Cloud container initializes using `copilot-setup-steps.yml`
5. **Implementation** — Agent reads codebase, plans approach, writes code
6. **Testing** — Agent runs your test suite, iterates on failures
7. **PR Opening** — Agent opens a pull request with description and changes
8. **Feedback Loop** — Agent responds to PR review comments and iterates

## Environment Configuration

### copilot-setup-steps.yml

This file tells the agent how to set up its cloud container. Place it at `.github/copilot-setup-steps.yml`:

```yaml
# .github/copilot-setup-steps.yml
# Defines the environment for Copilot Coding Agent

steps:
  # Install project dependencies
  - name: Install dependencies
    run: |
      npm install

  # Build the project to verify setup
  - name: Build project
    run: |
      npm run build

  # Run tests to establish baseline
  - name: Verify tests pass
    run: |
      npm test
```

For a Python project:

```yaml
# .github/copilot-setup-steps.yml
steps:
  - name: Set up Python environment
    run: |
      python -m venv .venv
      source .venv/bin/activate
      pip install -r requirements.txt

  - name: Run tests
    run: |
      source .venv/bin/activate
      pytest --tb=short
```

### Behavior Guidance with copilot-instructions.md

The agent reads `.github/copilot-instructions.md` for project-specific rules:

```markdown
<!-- .github/copilot-instructions.md -->

## Code Style
- Use TypeScript strict mode
- Prefer functional components with hooks
- Use named exports, not default exports

## Testing
- Write tests for all new functions
- Use Jest with React Testing Library
- Test files go in `__tests__/` next to source

## Architecture
- API routes in `src/api/`
- Shared types in `src/types/`
- Business logic in `src/services/`
```

## Writing Effective Issues

The quality of the agent's output depends heavily on issue quality.

### Good Issue Example

```markdown
## Add email validation to signup form

### Context
The signup form at `src/components/SignupForm.tsx` currently accepts
any string as an email. We need proper validation.

### Requirements
- Validate email format on blur and on submit
- Show inline error message below the email field
- Use the existing `FormError` component from `src/components/ui/`
- Prevent form submission if email is invalid
- Add unit tests in `src/components/__tests__/SignupForm.test.tsx`

### Acceptance Criteria
- [ ] Email regex validates standard formats (user@domain.com)
- [ ] Error message appears on blur with invalid email
- [ ] Form does not submit with invalid email
- [ ] Tests cover valid, invalid, and edge case emails
```

### Poor Issue Example (Avoid)

```markdown
Fix the signup form
```

### Issue Writing Best Practices

| Practice | Why It Matters |
|---|---|
| **Well-scoped** | Agent works best on single, focused tasks |
| **Clear acceptance criteria** | Agent knows when it's done |
| **Specific file hints** | Agent finds relevant code faster |
| **Reference existing patterns** | Agent follows your conventions |
| **Include test expectations** | Agent writes meaningful tests |

## How the Agent Iterates

The agent doesn't just write code once — it runs a feedback loop:

```
┌──────────────────────────────────────────┐
│          Agent Iteration Loop            │
├──────────────────────────────────────────┤
│                                          │
│   Write Code                             │
│       │                                  │
│       ▼                                  │
│   Run Tests ──── Pass ───▶ Open PR       │
│       │                                  │
│       │ Fail                             │
│       ▼                                  │
│   Analyze Errors                         │
│       │                                  │
│       ▼                                  │
│   Fix Code ─────────────▶ Run Tests      │
│                          (repeat up to   │
│                           N iterations)  │
└──────────────────────────────────────────┘
```

The agent:
- Reads test output and error messages
- Identifies the root cause of failures
- Modifies its implementation
- Re-runs tests to verify the fix
- Continues until tests pass or it reaches its iteration limit

## Limitations and When to Use

### Best For
- Bug fixes with clear reproduction steps
- Adding features to well-tested codebases
- Implementing patterns that exist elsewhere in the codebase
- Routine tasks: CRUD endpoints, form validation, utility functions

### Not Ideal For
- Architectural decisions or large refactors
- Tasks requiring deep domain knowledge not in the codebase
- Issues that depend on external services or manual testing
- Ambiguous requirements that need human judgment

### Key Limitations

```
┌──────────────────────────────────────────────┐
│           Coding Agent Boundaries            │
├──────────────────────────────────────────────┤
│                                              │
│  ✓ Reads your entire repository              │
│  ✓ Runs tests in cloud container             │
│  ✓ Creates branches and PRs                  │
│  ✓ Responds to review comments               │
│  ✓ Follows instruction files                 │
│                                              │
│  ✗ Cannot access external services           │
│  ✗ Cannot run GUI or browser tests           │
│  ✗ Cannot deploy or access production        │
│  ✗ Cannot ask clarifying questions           │
│  ✗ Works on one issue at a time per repo     │
│                                              │
└──────────────────────────────────────────────┘
```

## Configuration Checklist

Before using the Coding Agent, ensure your repository has:

```
your-repo/
├── .github/
│   ├── copilot-setup-steps.yml    # Environment setup
│   ├── copilot-instructions.md    # Behavior rules
│   └── ISSUE_TEMPLATE/
│       └── copilot-task.md        # Template for agent issues
├── package.json                    # (or equivalent) with test script
└── tests/                          # Working test suite
```

**Verify your setup:**

1. Your test suite runs and passes locally
2. `copilot-setup-steps.yml` installs all dependencies
3. Instructions cover your coding standards
4. Issue templates guide contributors to write agent-friendly issues
