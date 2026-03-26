# Multi-Agent Architecture

> Orchestration patterns, skills systems, and agent pipelines using AGENTS.md.

---

## What Is Multi-Agent Architecture?

Instead of one AI agent handling everything, multi-agent systems use **specialized agents** that collaborate:

```
┌─────────────────────────────────────────────┐
│              Orchestrator Agent               │
│        (routes tasks, manages flow)           │
├──────────┬──────────┬──────────┬────────────┤
│  Code    │  Test    │  Review  │  Deploy     │
│  Agent   │  Agent   │  Agent   │  Agent      │
│          │          │          │             │
│ Writes   │ Writes   │ Reviews  │ Handles     │
│ features │ tests    │ PRs      │ releases    │
└──────────┴──────────┴──────────┴────────────┘
```

---

## Pattern 1: Semantic Router

A routing layer classifies tasks and dispatches them to the right specialist agent.

```
User Task → [Router] → Code Agent / Test Agent / Docs Agent
```

### Implementation with AGENTS.md

```markdown
# AGENTS.md (root — acts as router context)

## Task Routing
- Tasks mentioning "test", "coverage", or "spec": → Test Engineer persona
- Tasks mentioning "docs", "README", "documentation": → Docs Agent persona
- Tasks mentioning "security", "vulnerability", "audit": → Security Agent persona
- All other coding tasks: → Code Engineer persona

## Personas
See .github/agents/ for per-persona instructions.
```

### When to Use
- Teams where different tasks need fundamentally different agent behavior
- Projects with dedicated test/docs/security workflows

---

## Pattern 2: Pipeline (Sequential)

Agents work in sequence, each building on the previous agent's output.

```
Issue → [Code Agent] → [Test Agent] → [Review Agent] → PR
```

### Implementation

Each agent has its own AGENTS.md or section:

```markdown
## Pipeline: Feature Implementation

### Stage 1: Code Agent
- Read the issue description
- Implement the feature
- Create necessary files
- Run lint: `pnpm lint`

### Stage 2: Test Agent  
- Read the implementation from Stage 1
- Write tests for all new functions
- Run tests: `pnpm test`
- Ensure 80% coverage

### Stage 3: Review Agent
- Review the complete changeset
- Check conventions compliance
- Verify tests cover edge cases
- Produce review summary
```

### When to Use
- Feature development workflows
- Ensuring quality gates (code → test → review)

---

## Pattern 3: Skills System

Agents expose modular **skills** — discrete functions that can be composed and chained.

```
skills/
├── test.md         — Run tests and report
├── lint.md         — Lint and auto-fix
├── deploy.md       — Deploy to staging
├── db-migrate.md   — Run database migrations
└── docs-build.md   — Build documentation
```

### Implementation

```markdown
# AGENTS.md

## Available Skills
- `test` — Run `pnpm test` and report pass/fail summary
- `lint` — Run `pnpm lint --fix` and list remaining issues  
- `typecheck` — Run `pnpm typecheck` and report errors
- `build` — Run `pnpm build` and verify success
- `deploy:staging` — Deploy to staging environment

## Skill Chaining
For feature PRs, run: test → lint → typecheck → build
For hotfix PRs, run: test → build → deploy:staging
```

### With GitHub Copilot Coding Agent

```markdown
## Workflow
When assigned an issue:
1. Read the issue description
2. Implement the changes
3. Run skill: test
4. Run skill: lint
5. If all pass, create PR
6. If any fail, fix and retry (max 3 attempts)
```

### When to Use
- Automating repetitive workflows
- Building CI-like quality gates into agent behavior
- Projects with multiple deployment targets

---

## Pattern 4: Parallel Agents

Multiple agents work simultaneously on independent tasks.

```
Issue: "Add user authentication"
          │
    ┌─────┼─────────┐
    ▼     ▼         ▼
  [DB]  [API]   [Frontend]
  Agent  Agent    Agent
    │     │         │
    ▼     ▼         ▼
  Schema  Routes   Login page
  + migration      + auth hooks
```

### Implementation

Per-directory AGENTS.md files enable parallel work:

```
services/
├── db/
│   ├── AGENTS.md     ← "Create user table migration"
├── api/
│   ├── AGENTS.md     ← "Create auth endpoints"
└── web/
    ├── AGENTS.md     ← "Create login page"
```

### When to Use
- Large features spanning multiple packages/services
- Monorepo with independent packages

---

## Pattern 5: Supervisor-Worker

A supervisor agent decomposes tasks and assigns sub-tasks to worker agents.

```
Supervisor (plans and coordinates)
    ├── Worker 1: "Implement User model"
    ├── Worker 2: "Create API endpoints"
    ├── Worker 3: "Write integration tests"
    └── Worker 4: "Update API documentation"
```

### Implementation

```markdown
# AGENTS.md

## Task Decomposition
For complex features (multi-file, multi-concern):
1. Break into independent sub-tasks
2. Execute sub-tasks in dependency order
3. Verify integration between sub-tasks
4. Run full test suite

## Sub-Task Template
Each sub-task should:
- Have a clear, single objective
- List files it will modify (and only those files)
- Include verification (test command or manual check)
- Be reviewable independently
```

### When to Use
- Complex features requiring coordinated changes
- Tasks that span 5+ files

---

## Agent Communication Patterns

### Direct File-Based Communication

Agents communicate through the codebase itself:
- Agent 1 creates a type definition
- Agent 2 imports and uses that type
- Agent 3 writes tests for the type

No explicit messaging needed — the code is the message.

### Status File Communication

For complex workflows, agents can use status files:

```
.agent-status/
├── feature-auth.json     ← "stage: testing, blocked: false"
├── feature-search.json   ← "stage: implementing, blocked: true, reason: needs API schema"
```

### Pull Request Communication

Agents communicate through PRs:
- Agent 1 creates PR for database changes
- Agent 2 reads the merged PR and builds API on top
- Agent 3 reads both PRs and builds the frontend

---

## Orchestration with GitHub Actions

```yaml
# .github/workflows/agent-pipeline.yml
name: Agent Pipeline
on:
  issues:
    types: [assigned]

jobs:
  implement:
    if: github.event.assignee.login == 'copilot[bot]'
    runs-on: ubuntu-latest
    steps:
      - name: Implement feature
        uses: github/copilot-coding-agent@v1
        with:
          task: implement

  test:
    needs: implement
    runs-on: ubuntu-latest
    steps:
      - name: Generate tests
        uses: github/copilot-coding-agent@v1
        with:
          task: write-tests

  review:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Code review
        uses: github/copilot-coding-agent@v1
        with:
          task: review
```

---

## Choosing the Right Pattern

| Pattern | Best For | Complexity |
|---------|----------|-----------|
| Semantic Router | Different task types | Low |
| Pipeline | Quality gates | Medium |
| Skills System | Reusable automation | Medium |
| Parallel Agents | Independent packages | High |
| Supervisor-Worker | Complex features | High |

**Start with the simplest pattern that solves your problem.** Add complexity only when needed.

---

*Next: [Enterprise & Org Strategies](enterprise-strategies.md) →*
