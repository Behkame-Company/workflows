# Agentic Workflow Templates

> Ready-to-use templates for configuring agent behavior — copy, customize, and ship.

---

## 1. copilot-instructions.md — Agent Mode Configuration

Place at `.github/copilot-instructions.md` to configure GitHub Copilot across your repository.

```markdown
# Copilot Instructions

## Build & Test Commands
- Build: `npm run build`
- Test all: `npm test`
- Test single: `npm test -- --grep "test name"`
- Lint: `npm run lint`
- Type check: `npx tsc --noEmit`

## Code Conventions
- Language: TypeScript (strict mode)
- Formatting: Prettier with project config
- Imports: Use path aliases (`@/` for `src/`)
- Naming: camelCase for variables/functions, PascalCase for types/classes
- Error handling: Always use typed errors, never throw raw strings
- Tests: Co-locate with source files as `*.test.ts`

## Architecture Rules
- All API routes go in `src/routes/`
- Business logic in `src/services/` — never in route handlers
- Database queries only in `src/repositories/`
- Shared types in `src/types/`

## Forbidden Patterns
- Do NOT use `any` type — use `unknown` with type guards
- Do NOT use `console.log` — use the logger from `src/lib/logger`
- Do NOT modify test files to make tests pass
- Do NOT commit `.env` files or secrets
- Do NOT use synchronous file I/O in request handlers

## Testing Requirements
- All new functions must have tests
- Test edge cases: null, undefined, empty arrays, boundary values
- Mock external services, never call real APIs in tests
- Run full test suite before reporting completion
```

## 2. CLAUDE.md — Claude Code / Agentic Workflows

Place at the repository root for Claude Code and similar agentic tools.

```markdown
# CLAUDE.md

## Project Context
This is a Node.js REST API using Express, TypeScript, and PostgreSQL.
The project serves as the backend for a task management application.

## Commands
- Install: `npm ci`
- Build: `npm run build`
- Test: `npm test`
- Test (watch): `npm run test:watch`
- Lint: `npm run lint -- --fix`
- Dev server: `npm run dev`
- Database migrations: `npm run migrate`

## Code Conventions
- Use functional style over classes where possible
- Prefer `const` over `let`, never use `var`
- Use early returns to reduce nesting
- All async functions must have error handling
- Use zod for runtime validation at API boundaries

## Agent Behavior Rules
- Always run tests after making changes
- Read existing code before writing new code
- Follow existing patterns in the codebase
- Make minimal, focused changes
- Commit after each logical unit of work
- Never modify generated files (*.gen.ts, migrations)
```

## 3. copilot-setup-steps.yml — Codespace/Agent Environment

Place at `.github/copilot-setup-steps.yml` for automated environment setup.

```yaml
# .github/copilot-setup-steps.yml
# Configures the environment for Copilot coding agent

steps:
  # Install project dependencies
  - name: Install dependencies
    run: npm ci

  # Build the project
  - name: Build project
    run: npm run build

  # Set up test database
  - name: Start test database
    run: |
      docker compose -f docker-compose.test.yml up -d
      npm run migrate:test

  # Verify environment
  - name: Verify setup
    run: |
      npm test -- --bail --timeout 10000
      echo "Environment ready for agent"
```

## 4. .prompt.md — Reusable Agent Tasks

Place in `.github/prompts/` for task-specific agent instructions.

```markdown
---
tools:
  - name: terminal
  - name: file_editor
  - name: code_search
description: "Add a new API endpoint with tests"
---

# Add API Endpoint

## Steps
1. Read the existing route files in `src/routes/` to understand patterns
2. Create the new route handler following existing conventions
3. Add request validation using zod schemas
4. Create the service layer function in `src/services/`
5. Add repository layer if database access is needed
6. Write tests covering:
   - Happy path
   - Validation errors (400)
   - Not found cases (404)
   - Server errors (500)
7. Register the route in `src/routes/index.ts`
8. Run the full test suite: `npm test`
9. Run the linter: `npm run lint`
```

### Additional Prompt File: Fix a Bug

```markdown
---
tools:
  - name: terminal
  - name: file_editor
  - name: code_search
description: "Systematic bug fix workflow"
---

# Fix Bug

## Steps
1. Reproduce the bug — write a failing test first
2. Read the relevant source code to understand the logic
3. Identify the root cause (not just the symptom)
4. Implement the minimal fix
5. Verify the failing test now passes
6. Run the full test suite to check for regressions
7. Check for similar patterns elsewhere that might have the same bug
```

## 5. Skill Folder Template

Structure for a custom Copilot skill in `.github/skills/`:

```
.github/skills/deploy-preview/
├── instructions.md
└── scripts/
    ├── deploy.sh
    └── check-status.sh
```

```markdown
<!-- .github/skills/deploy-preview/instructions.md -->
# Deploy Preview Skill

## Description
Deploy a preview environment for the current branch.

## When to Use
Use this skill when the user asks to deploy, preview, or stage their changes.

## Steps
1. Run `scripts/deploy.sh` with the current branch name
2. Wait for deployment to complete (check with `scripts/check-status.sh`)
3. Return the preview URL to the user

## Environment Requirements
- `DEPLOY_TOKEN` must be set in the environment
- Branch must have a clean build (run `npm run build` first)

## Error Handling
- If build fails, report the error and do not deploy
- If deployment times out (>5 minutes), report and provide manual steps
```

```bash
#!/bin/bash
# .github/skills/deploy-preview/scripts/deploy.sh
set -euo pipefail

BRANCH="${1:-$(git branch --show-current)}"
echo "Deploying preview for branch: $BRANCH"

# Build first
npm run build

# Deploy to preview environment
npx vercel deploy --prebuilt --token="$DEPLOY_TOKEN" 2>&1
```

## 6. Custom Agent Configuration Template

For defining custom agents (e.g., in `.github/agents/`):

```markdown
<!-- .github/agents/code-reviewer.md -->
# Code Reviewer Agent

## Role
You are a senior code reviewer. Review changes for correctness, 
security, and adherence to project conventions.

## Tools Available
- `terminal` — run commands (tests, linting)
- `file_editor` — read files (do NOT modify)
- `code_search` — find related code

## Review Checklist
1. **Correctness**: Does the code do what it claims?
2. **Tests**: Are changes covered by tests?
3. **Security**: Any new attack vectors?
4. **Performance**: Any obvious bottlenecks?
5. **Conventions**: Follows project style?

## Output Format
Produce a structured review:
- 🔴 **Critical** — Must fix before merge
- 🟡 **Suggestion** — Should consider
- 🟢 **Praise** — Good patterns to reinforce

## Rules
- Do NOT modify any files
- Focus on substance, not style
- If tests don't pass, flag as critical
- Be specific: reference file and line numbers
```

## 7. Multi-Agent Orchestration Script

Template for coordinating multiple agents on a feature:

```typescript
// orchestrate-feature.ts
// Template for multi-agent feature development

interface AgentTask {
  name: string;
  prompt: string;
  dependencies: string[];
  files: string[];  // Files this agent may modify
}

const featurePlan: AgentTask[] = [
  {
    name: "schema",
    prompt: "Create database migration for the new 'comments' table",
    dependencies: [],
    files: ["src/migrations/", "src/types/comment.ts"],
  },
  {
    name: "backend",
    prompt: "Implement CRUD API for comments",
    dependencies: ["schema"],
    files: ["src/routes/comments.ts", "src/services/comment-service.ts"],
  },
  {
    name: "frontend",
    prompt: "Build comment component with list and form",
    dependencies: ["schema"],  // Can run parallel with backend
    files: ["src/components/Comments/"],
  },
  {
    name: "tests",
    prompt: "Write integration tests for the comments feature",
    dependencies: ["backend", "frontend"],
    files: ["src/__tests__/comments.test.ts"],
  },
];

async function orchestrate(tasks: AgentTask[]) {
  const completed = new Set<string>();
  
  while (completed.size < tasks.length) {
    // Find tasks whose dependencies are met
    const ready = tasks.filter(
      t => !completed.has(t.name) 
        && t.dependencies.every(d => completed.has(d))
    );
    
    // Execute ready tasks in parallel
    const results = await Promise.all(
      ready.map(task => executeAgent(task))
    );
    
    for (const result of results) {
      if (result.success) completed.add(result.taskName);
      else throw new Error(`Task ${result.taskName} failed: ${result.error}`);
    }
  }
}
```

## 8. Agent Eval Suite JSON Template

Template for evaluating agent performance across tasks:

```json
{
  "name": "code-agent-eval-suite",
  "version": "1.0.0",
  "description": "Evaluation suite for coding agent capabilities",
  "config": {
    "timeout_seconds": 300,
    "max_tokens": 100000,
    "parallel": false
  },
  "evaluations": [
    {
      "id": "simple-bug-fix",
      "category": "bug-fix",
      "difficulty": "easy",
      "description": "Fix an off-by-one error in array processing",
      "setup": {
        "files": {
          "src/utils.ts": "export function getLastN<T>(arr: T[], n: number): T[] {\n  return arr.slice(arr.length - n - 1);\n}",
          "src/utils.test.ts": "import { getLastN } from './utils';\ntest('returns last 2 items', () => {\n  expect(getLastN([1,2,3,4,5], 2)).toEqual([4,5]);\n});"
        },
        "command": "npm install"
      },
      "task": "Fix the failing test in src/utils.test.ts by correcting the bug in src/utils.ts",
      "verification": {
        "command": "npm test",
        "expected_exit_code": 0,
        "file_checks": [
          {
            "path": "src/utils.ts",
            "must_contain": "arr.length - n",
            "must_not_contain": "arr.length - n - 1"
          },
          {
            "path": "src/utils.test.ts",
            "must_not_be_modified": true
          }
        ]
      },
      "scoring": {
        "correct_fix": 5,
        "tests_pass": 3,
        "no_test_modification": 2,
        "max_score": 10
      }
    },
    {
      "id": "add-feature",
      "category": "feature",
      "difficulty": "medium",
      "description": "Add input validation to an existing endpoint",
      "setup": {
        "repository": "https://github.com/example/eval-repo",
        "branch": "eval-validation",
        "command": "npm ci"
      },
      "task": "Add zod validation to POST /api/users endpoint. Validate: email (valid format), name (1-100 chars), age (optional, 0-150)",
      "verification": {
        "command": "npm test",
        "expected_exit_code": 0,
        "endpoint_tests": [
          { "method": "POST", "path": "/api/users", "body": {}, "expected_status": 400 },
          { "method": "POST", "path": "/api/users", "body": {"email": "bad"}, "expected_status": 400 },
          { "method": "POST", "path": "/api/users", "body": {"email": "a@b.com", "name": "Test"}, "expected_status": 201 }
        ]
      },
      "scoring": {
        "validation_works": 4,
        "tests_added": 3,
        "edge_cases": 2,
        "clean_code": 1,
        "max_score": 10
      }
    }
  ],
  "aggregate_scoring": {
    "pass_threshold": 0.7,
    "categories": ["bug-fix", "feature", "refactor", "security"],
    "report_format": "markdown"
  }
}
```

## Customization Tips

✅ **Do:**
- Start with the template closest to your stack
- Remove sections that don't apply
- Add project-specific commands and conventions
- Keep instructions under 15 key rules (agents ignore long lists)
- Test templates with real tasks before committing

❌ **Don't:**
- Copy templates verbatim without customization
- Include contradictory instructions
- Add rules you can't verify or enforce
- Over-specify — let agents use judgment for routine decisions
