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

```markdown
# AGENTS.md

## Available Skills
- `test` — Run `pnpm test` and report pass/fail summary
- `lint` — Run `pnpm lint --fix` and list remaining issues
- `build` — Run `pnpm build` and verify success

## Skill Chaining
For feature PRs, run: test → lint → typecheck → build
For hotfix PRs, run: test → build → deploy:staging
```

### When to Use
- Automating repetitive workflows and CI-like quality gates
- Projects with multiple deployment targets

---

## Pattern 4: Parallel Agents

Multiple agents work simultaneously on independent tasks, each with its own per-directory AGENTS.md.

```
services/
├── db/AGENTS.md      ← "Create user table migration"
├── api/AGENTS.md     ← "Create auth endpoints"
└── web/AGENTS.md     ← "Create login page"
```

### When to Use
- Large features spanning multiple packages/services
- Monorepo with independent packages

---

## Pattern 5: Supervisor-Worker

A supervisor agent decomposes complex tasks and assigns sub-tasks to worker agents in dependency order.

```markdown
# AGENTS.md

## Task Decomposition
For complex features (multi-file, multi-concern):
1. Break into independent sub-tasks (each with single objective)
2. Execute in dependency order
3. Verify integration between sub-tasks
4. Run full test suite
```

### When to Use
- Complex features requiring coordinated changes across 5+ files

---

## Agent Communication Patterns

- **File-based**: Agents communicate through the codebase itself (types, imports, tests)
- **Status files**: For complex workflows, use `.agent-status/*.json` to track stage and blockers
- **Pull requests**: Sequential agents build on each other’s merged PRs

---

## Orchestration with GitHub Actions

Agent pipelines can be orchestrated as sequential GitHub Actions jobs (implement → test → review), triggered on issue assignment to `copilot[bot]`. Each job invokes the coding agent with a specific task.

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
