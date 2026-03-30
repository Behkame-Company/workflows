# Agent Composition

> Combining skills and agents into powerful workflows — skill stacking, agent pipelines, delegation patterns, and building composable systems that exceed what any single agent can do.

---

## Why Compose?

Individual skills and agents solve individual problems. Composition solves **workflows** — multi-step processes that span multiple concerns.

```
┌──────────────────────────────────────────────────────────────┐
│        Single Agent vs Composed System                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Single Agent                 Composed System               │
│   ────────────                 ───────────────               │
│   One skill set                Multiple specialties          │
│   One context                  Fresh context per step        │
│   Generic knowledge            Deep expertise per agent      │
│                                                              │
│   "Review this PR"             Security Agent ──▶            │
│   (surface-level review)       Style Agent    ──▶  Merged    │
│                                Test Agent     ──▶  Review    │
│                                (deep, expert review)         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Composition Pattern 1: Skill Stacking

An agent loads multiple skills to handle a complex task that spans concerns.

```
┌──────────────────────────────────────────────────────────────┐
│              Skill Stacking                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Task: "Deploy the new user service"                        │
│                                                              │
│   Agent loads:                                               │
│   ┌──────────────────┐                                       │
│   │ deploy skill     │  How to deploy                        │
│   ├──────────────────┤                                       │
│   │ db-migration     │  Run schema changes first             │
│   │ skill            │                                       │
│   ├──────────────────┤                                       │
│   │ monitoring skill │  Set up alerts after deploy           │
│   └──────────────────┘                                       │
│                                                              │
│   Agent follows all three skills' instructions               │
│   in the right order to complete the task                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Example: Deployment with Migration

```markdown
<!-- Agent receives: "Deploy v2.3.0 with the new orders table" -->

<!-- Agent loads deploy skill + db-migration skill -->

## Execution Plan:
1. Run database migration (from db-migration skill)
   - Create orders table migration
   - Test rollback
   - Apply to staging
2. Deploy application (from deploy skill)
   - Run pre-deploy checks
   - Bump version to 2.3.0
   - Tag and push
3. Verify deployment (from deploy skill)
   - Health check
   - Smoke tests
```

## Composition Pattern 2: Agent Pipelines

Output of one agent feeds into the next. Each agent transforms the work.

```
┌──────────────────────────────────────────────────────────────┐
│              Agent Pipeline                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Input ──▶ Agent A ──▶ Agent B ──▶ Agent C ──▶ Output       │
│             (spec)      (implement)  (review)                │
│                                                              │
│   Example: Feature Pipeline                                  │
│                                                              │
│   ┌─────────┐    ┌────────────┐    ┌──────────┐             │
│   │  Spec   │    │   Code     │    │  Review  │             │
│   │  Agent  │───▶│   Agent    │───▶│  Agent   │             │
│   │         │    │            │    │          │             │
│   │ Writes  │    │ Implements │    │ Reviews  │             │
│   │ spec +  │    │ against    │    │ for bugs │             │
│   │ types   │    │ the spec   │    │ + style  │             │
│   └─────────┘    └────────────┘    └──────────┘             │
│                                                              │
│   Each agent has focused context and expertise               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Pipeline Implementation

```bash
# Step 1: Spec agent creates the specification
copilot -p "Read the requirements in issue #42 and create a
technical spec in docs/specs/webhook-system.md. Include
interface definitions in src/types/webhook.ts." --allow-all-tools

# Step 2: Implementation agent builds against the spec
copilot -p "Implement the webhook system according to the spec
in docs/specs/webhook-system.md. Use the types from
src/types/webhook.ts. Write tests." --allow-all-tools

# Step 3: Review agent checks the implementation
copilot -p "Review the webhook implementation against the spec
in docs/specs/webhook-system.md. Check for: missing error
handling, untested paths, spec compliance." --allow-all-tools
```

## Composition Pattern 3: Agent Delegation

An orchestrator agent delegates sub-tasks to specialized agents.

```
┌──────────────────────────────────────────────────────────────┐
│              Agent Delegation                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                 ┌──────────────┐                              │
│                 │ Orchestrator │                              │
│                 │    Agent     │                              │
│                 └──────┬───────┘                              │
│                        │                                     │
│           ┌────────────┼────────────┐                        │
│           │            │            │                        │
│           ▼            ▼            ▼                        │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│   │ Frontend │  │   API    │  │ Database │                  │
│   │ Expert   │  │ Designer │  │ Specialist│                  │
│   │          │  │          │  │          │                  │
│   │ Builds   │  │ Creates  │  │ Creates  │                  │
│   │ UI       │  │ endpoints│  │ schema   │                  │
│   └──────────┘  └──────────┘  └──────────┘                  │
│                        │                                     │
│                        ▼                                     │
│               Orchestrator integrates                         │
│               and runs E2E tests                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Delegation in Practice

```bash
# Orchestrator session (Window 1)
$ copilot
> I need a new "teams" feature. Plan the work and delegate:
> - Frontend: new TeamList and TeamDetail components
> - API: CRUD endpoints at /api/teams
> - Database: teams and team_members tables
>
> Create interface contracts first, then I'll spin up
> worker sessions for each.

# After orchestrator creates interfaces...

# Frontend session (Window 2)
$ copilot -p "Implement the frontend for the teams feature.
Interface contracts are in src/types/team.ts.
Follow the component patterns in src/components/ui/.
Create TeamList and TeamDetail components with tests."

# API session (Window 3)
$ copilot -p "Implement CRUD endpoints for teams at /api/teams.
Use the types from src/types/team.ts.
Follow the route patterns in src/app/api/."

# Database session (Window 4)
$ copilot -p "Create database migration for teams and team_members.
Schema defined in docs/specs/teams.md.
Follow migration patterns in migrations/."
```

## Composition Pattern 4: Skill Inheritance

Personal skills provide defaults; project skills specialize them.

```
┌──────────────────────────────────────────────────────────────┐
│              Skill Inheritance                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   ~/.agents/skills/code-review/                              │
│   ─────────────────────────────                              │
│   Generic review checklist:                                  │
│   • Check error handling                                     │
│   • Check resource cleanup                                   │
│   • Check input validation                                   │
│                                                              │
│        ▼ (project skill extends)                             │
│                                                              │
│   .agents/skills/code-review/                                │
│   ───────────────────────────                                │
│   Project-specific additions:                                │
│   • Check Prisma query efficiency                            │
│   • Verify audit logging on mutations                        │
│   • Ensure HIPAA compliance for PII fields                   │
│                                                              │
│   Agent sees: generic rules + project-specific rules         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Building Composable Agents

### Principle 1: Keep Agents Focused

```
Good:                              Bad:
┌──────────────┐                   ┌──────────────────────────┐
│ Security     │                   │ Everything Agent          │
│ Reviewer     │                   │                          │
│              │                   │ Reviews security         │
│ Only checks  │                   │ AND writes code          │
│ security     │                   │ AND deploys              │
│ issues       │                   │ AND writes docs          │
└──────────────┘                   └──────────────────────────┘
```

### Principle 2: Define Clear Interfaces

Agents communicate through structured artifacts, not conversation:

```typescript
// Structured output for agent-to-agent communication
interface AgentOutput {
  status: "success" | "failure" | "needs-review";
  artifacts: string[];     // Files created or modified
  summary: string;         // What was done
  issues: Issue[];         // Problems found
  nextSteps: string[];     // Recommended follow-ups
}
```

### Principle 3: Use Structured Output

```json
// Agent A writes structured output
{
  "review": {
    "critical": [
      {
        "file": "src/auth/login.ts",
        "line": 42,
        "issue": "SQL injection via unsanitized email parameter",
        "fix": "Use parameterized query"
      }
    ],
    "suggestions": [],
    "approved": false
  }
}

// Agent B reads and acts on the structured output
```

## Composition Architecture

```
┌──────────────────────────────────────────────────────────────┐
│           Composable Agent Architecture                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Skills Layer                                               │
│   ────────────                                               │
│   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐              │
│   │ Deploy │ │ Review │ │ Test   │ │ Migrate│              │
│   │ Skill  │ │ Skill  │ │ Skill  │ │ Skill  │              │
│   └────┬───┘ └───┬────┘ └───┬────┘ └───┬────┘              │
│        │         │          │          │                    │
│   Agent Layer                                                │
│   ───────────                                                │
│   ┌────────────────────────────────────────────┐             │
│   │            Specialized Agents              │             │
│   │  Frontend │ API │ Database │ Security       │             │
│   └──────────────────┬─────────────────────────┘             │
│                      │                                       │
│   Orchestration Layer                                        │
│   ───────────────────                                        │
│   ┌──────────────────┴───────────────────────┐               │
│   │  Pipeline  │  Delegator  │  Parallel     │               │
│   └──────────────────────────────────────────┘               │
│                                                              │
│   Coordination: Files, structured output, tracking docs      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Example: Complete Code Review Workflow

Compose three skills into a comprehensive review:

```bash
# The composed review workflow
BRANCH=$(git branch --show-current)

# Security review
copilot -p "Using the security-review skill, review all changes
in $BRANCH compared to main. Focus exclusively on security issues.
Output as JSON to .review/security.json" --allow-all-tools

# Code quality review
copilot -p "Using the code-review skill, review all changes
in $BRANCH compared to main. Focus on bugs and logic errors.
Output as JSON to .review/quality.json" --allow-all-tools

# Test coverage review
copilot -p "Using the test-coverage skill, check that all new
code in $BRANCH has adequate test coverage. Identify untested
paths. Output as JSON to .review/coverage.json" --allow-all-tools

# Synthesize reviews
copilot -p "Read all review files in .review/ and create a
unified review summary. Prioritize by severity. Output to
.review/summary.md" --allow-all-tools
```

## When Not to Compose

Composition adds complexity. Use it only when needed:

| Scenario | Compose? | Why |
|---|---|---|
| Simple bug fix | No | Single agent is sufficient |
| New component | No | One skill handles it |
| Full feature across stack | Yes | Multiple specialties needed |
| Complex deployment | Yes | Multiple skills, verification |
| Quick code review | No | Single agent review is fine |
| Audit across multiple concerns | Yes | Each concern needs depth |
