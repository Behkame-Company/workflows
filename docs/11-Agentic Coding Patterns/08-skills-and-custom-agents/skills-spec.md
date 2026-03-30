# Agent Skills Specification

> The open standard for agent capabilities — a cross-platform specification that defines how AI coding agents discover, load, and execute reusable skills.

---

## What Is the Agent Skills Spec?

The agentskills specification defines a universal format for packaging agent knowledge. Skills built to this spec work across AI systems — Copilot, Claude Code, and any other tool that implements the standard.

```
┌──────────────────────────────────────────────────────────────┐
│              Agent Skills Specification                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Skill = Folder with structured contents                    │
│                                                              │
│   ┌────────────────────────────────┐                         │
│   │  instructions.md               │  How to do the task     │
│   │  (required)                    │                         │
│   ├────────────────────────────────┤                         │
│   │  scripts/                      │  Automation helpers     │
│   │  (optional)                    │                         │
│   ├────────────────────────────────┤                         │
│   │  resources/                    │  Reference data         │
│   │  (optional)                    │                         │
│   └────────────────────────────────┘                         │
│                                                              │
│   Open Standard ── Works Across AI Systems                   │
│   agentskills/agentskills on GitHub                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Skill Locations

Skills live at different scopes, from project-level (shared with team) to personal (your machine only).

```
┌──────────────────────────────────────────────────────────────┐
│                    Skill Location Hierarchy                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   PROJECT LEVEL (committed to git)                           │
│   ────────────────────────────────                           │
│   .github/skills/         GitHub / Copilot convention        │
│   .claude/skills/         Claude Code convention             │
│   .agents/skills/         Cross-platform convention          │
│                                                              │
│   PERSONAL LEVEL (local to your machine)                     │
│   ──────────────────────────────────────                     │
│   ~/.copilot/skills/      Your Copilot skills                │
│   ~/.claude/skills/       Your Claude Code skills            │
│   ~/.agents/skills/       Your cross-platform skills         │
│                                                              │
│   COMING SOON                                                │
│   ───────────                                                │
│   Organization-level      Shared across org repos            │
│   Enterprise-level        Shared across enterprise           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Which Location Should You Use?

| Location | Use When |
|---|---|
| `.agents/skills/` | Skill works with any AI system |
| `.github/skills/` | Team uses Copilot primarily |
| `.claude/skills/` | Team uses Claude Code primarily |
| `~/.agents/skills/` | Personal skill for all projects |
| `~/.copilot/skills/` | Personal Copilot-specific skill |

## Skill Structure

### Minimal Skill (Instructions Only)

```
my-skill/
└── instructions.md
```

### Full Skill

```
my-skill/
├── instructions.md           # Required: task procedure
├── scripts/                  # Optional: automation
│   ├── setup.sh              #   Environment preparation
│   ├── validate.sh           #   Output validation
│   └── cleanup.sh            #   Post-task cleanup
└── resources/                # Optional: reference data
    ├── template.ts           #   Code template
    ├── checklist.md          #   Review checklist
    └── examples/             #   Example inputs/outputs
        ├── input.json
        └── expected-output.json
```

### Directory Layout Diagram

```
┌──────────────────────────────────────────────────┐
│              Skill Directory Layout               │
├──────────────────────────────────────────────────┤
│                                                   │
│  project-root/                                    │
│  ├── .agents/                                     │
│  │   └── skills/                                  │
│  │       ├── deploy/                              │
│  │       │   ├── instructions.md                  │
│  │       │   ├── scripts/                         │
│  │       │   │   └── pre-deploy.sh                │
│  │       │   └── resources/                       │
│  │       │       └── checklist.md                 │
│  │       ├── db-migration/                        │
│  │       │   ├── instructions.md                  │
│  │       │   └── resources/                       │
│  │       │       └── migration-template.sql       │
│  │       └── code-review/                         │
│  │           └── instructions.md                  │
│  ├── src/                                         │
│  ├── tests/                                       │
│  └── package.json                                 │
│                                                   │
└──────────────────────────────────────────────────┘
```

## The instructions.md File

The heart of every skill. It tells the agent **how** to perform the task.

### Required Elements

```markdown
# [Skill Name]

## When to Use
[Describe the trigger conditions — when should the agent activate this skill]

## Process
[Step-by-step procedure the agent should follow]

### Step 1: [Name]
[Details]

### Step 2: [Name]
[Details]

## Rules
[Hard constraints the agent must follow]

## Examples
[Input/output examples showing expected behavior]
```

### instructions.md Best Practices

| Practice | Reasoning |
|---|---|
| Start with "When to Use" | Helps agent decide if skill is relevant |
| Number every step | Agent follows ordered sequences reliably |
| Include exact commands | No ambiguity — agent runs exactly this |
| List explicit rules | Hard boundaries the agent must not cross |
| Add examples | Models learn patterns better from examples |

## How Agents Discover Skills

Discovery is automatic. When an agent receives a task, it scans skill directories for relevant matches:

```
┌──────────────────────────────────────────────────┐
│           Skill Discovery Process                 │
├──────────────────────────────────────────────────┤
│                                                   │
│  Agent Receives Task                              │
│       │                                           │
│       ▼                                           │
│  Scan Skill Directories                           │
│  ├── .github/skills/*                             │
│  ├── .claude/skills/*                             │
│  ├── .agents/skills/*                             │
│  ├── ~/.copilot/skills/*                          │
│  ├── ~/.claude/skills/*                           │
│  └── ~/.agents/skills/*                           │
│       │                                           │
│       ▼                                           │
│  Match by Relevance                               │
│  • Skill name matches task keywords               │
│  • instructions.md "When to Use" matches          │
│       │                                           │
│       ▼                                           │
│  Load Matched Skills                              │
│  • instructions.md added to agent context         │
│  • Scripts and resources available for use         │
│       │                                           │
│       ▼                                           │
│  Agent Executes Following Skill Instructions      │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Cross-Platform Compatibility

Skills in `.agents/skills/` work across all compliant AI systems:

```
┌──────────────────────────────────────────────────┐
│         Cross-Platform Skill Usage                │
├──────────────────────────────────────────────────┤
│                                                   │
│              .agents/skills/deploy/               │
│                       │                           │
│           ┌───────────┼───────────┐               │
│           ▼           ▼           ▼               │
│       Copilot    Claude Code    Other             │
│       CLI        CLI            Compliant         │
│                                 Agents            │
│                                                   │
│  Same instructions, same behavior, any tool       │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Example: Complete Skill

```
.agents/skills/new-api-endpoint/
├── instructions.md
├── scripts/
│   └── generate-scaffold.sh
└── resources/
    ├── route-template.ts
    └── test-template.ts
```

**instructions.md:**

```markdown
# New API Endpoint Skill

## When to Use
When asked to create a new API endpoint, REST route, or HTTP handler.

## Process

### Step 1: Gather Requirements
Identify from the request:
- HTTP method (GET, POST, PUT, DELETE)
- Resource path (/api/[resource])
- Request body schema (if applicable)
- Response shape
- Authentication requirements

### Step 2: Create Route File
Create `src/app/api/[resource]/route.ts` using the template
in resources/route-template.ts.

### Step 3: Define Validation Schema
Add Zod schema for request validation in the route file.

### Step 4: Implement Handler
- Use the service layer for business logic
- Handle errors with try/catch
- Return consistent response format: { data } or { error }

### Step 5: Write Tests
Create test file using resources/test-template.ts as a starting point.
Test: success case, validation errors, not found, server errors.

### Step 6: Update Types
Add route types to src/types/api.ts.

### Step 7: Verify
Run: `npm test -- --testPathPattern=[resource]`
Run: `npm run lint`
Run: `npx tsc --noEmit`

## Rules
- Never skip validation
- Never return raw database errors to clients
- Always include tests for error paths
- Follow existing patterns in src/app/api/
```
