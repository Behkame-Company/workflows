# Cross-Team Consistency

> Maintaining AI standards across multiple teams without killing team autonomy. The goal is shared foundations with team-level customization — not rigid uniformity.

---

## The Challenge

As AI adoption scales across teams, each team naturally develops its own practices:

```
Team A: "We use detailed instruction files with strict rules"
Team B: "We prefer minimal instruction files and flexible prompts"
Team C: "What's an instruction file?"
Team D: "We created our own AI review checklist"
Team E: "We just use whatever works"
```

Without coordination, you get:
- **Inconsistent quality** across teams
- **Developers switching teams** have to relearn AI practices
- **Shared code** (libraries, services) gets conflicting AI-generated styles
- **Security gaps** where some teams have guardrails and others don't
- **Duplicated effort** as each team solves the same problems independently

---

## The Governance Model

### Two-Layer Architecture

```
┌──────────────────────────────────────────────────┐
│              Organization Level                   │
│  (Central AI Platform Team)                       │
│                                                   │
│  • Security rules (mandatory)                     │
│  • Core quality standards (mandatory)             │
│  • Shared prompt library (recommended)            │
│  • Tool configuration baseline (mandatory)        │
│  • Review process requirements (mandatory)        │
└──────────────────────────┬───────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Team A     │  │   Team B     │  │   Team C     │
│              │  │              │  │              │
│ + React rules│  │ + Python     │  │ + Go         │
│ + UI prompts │  │   rules      │  │   rules      │
│ + Component  │  │ + ML prompts │  │ + API prompts│
│   patterns   │  │ + Notebook   │  │ + gRPC       │
│              │  │   patterns   │  │   patterns   │
└──────────────┘  └──────────────┘  └──────────────┘

Organization rules = mandatory base
Team rules = additive customization
```

### What Lives at Each Level

**Organization Level (Mandatory)**

| Standard | Rationale |
|---|---|
| Security rules | Consistent security posture across all code |
| Approved AI tools | License management, data residency |
| Review process | Every AI PR gets human review |
| Secret scanning | No credentials in any repository |
| Quality gates | Minimum test coverage, linting, SAST |
| Data handling | What can/cannot be shared with AI tools |

**Team Level (Customizable)**

| Standard | Examples |
|---|---|
| Naming conventions | camelCase vs. snake_case per language |
| File organization | Project-specific structure |
| Framework patterns | React, Django, Go patterns |
| Prompt library | Team-specific prompt templates |
| Additional quality rules | Stricter coverage, extra linting |
| Custom skills | Team-specific AI automation |

---

## Implementation Strategies

### 1. Organization-Level Instruction File

Create a base instruction file that all teams inherit:

```markdown
<!-- .github/copilot-instructions-base.md (org-level template) -->

# Organization AI Standards

## Security (Mandatory — All Teams)
- Never generate code containing hardcoded secrets
- All database queries must use parameterized statements
- User input must be validated before processing
- Output must be sanitized to prevent XSS
- No eval() or Function() constructor usage

## Quality (Mandatory — All Teams)
- All new code must have tests
- AI-generated code requires human review
- Existing tests must not be broken
- SAST scan must pass

## Process (Mandatory — All Teams)
- PRs must use the AI disclosure template
- Minimum 1 human reviewer for AI-generated code
- Minimum 2 reviewers for security-sensitive code
```

Each team then extends with team-specific rules:

```markdown
<!-- .github/copilot-instructions.md (team-level) -->

# Team Frontend Standards

## Inherits: Organization AI Standards

## TypeScript Conventions
- Strict mode enabled
- No any types
- Interface naming: no I prefix

## React Patterns
- Functional components only
- Hooks at the top of component
- CSS modules for styling

## Testing
- Vitest + Testing Library
- Coverage threshold: 85%
```

### 2. Shared Prompt Library

Maintain a central prompt repository with team-specific extensions:

```
org-prompts/                    (central repository)
├── security/
│   ├── security-review.prompt.md
│   └── auth-check.prompt.md
├── testing/
│   ├── unit-tests.prompt.md
│   └── integration-tests.prompt.md
└── review/
    └── code-review.prompt.md

team-frontend-prompts/          (team repository)
├── react/
│   ├── create-component.prompt.md
│   └── create-hook.prompt.md
└── testing/
    └── rtl-component-test.prompt.md
```

### 3. Cross-Team Reviews

Periodic reviews ensure consistency:

**Monthly: Cross-Team Instruction File Audit**
```markdown
## Cross-Team Audit Checklist
- [ ] All teams include org-level security rules
- [ ] No teams have conflicting org-level rules
- [ ] Team-specific rules don't weaken org standards
- [ ] New patterns from one team could benefit others
```

**Quarterly: Cross-Team Prompt Library Review**
```markdown
## Prompt Library Review
- [ ] Central prompts are up to date
- [ ] Team prompts don't duplicate central ones
- [ ] Team-specific prompts that could be promoted to central
- [ ] Deprecated prompts removed
```

### 4. Inner-Source AI Tools

Teams that create useful AI tools (skills, agents, MCP servers) share them with the organization:

```
Inner-source AI tooling:
├── Shared skills (available to all teams)
│   ├── code-review-skill
│   ├── test-generation-skill
│   └── migration-assistant-skill
├── Team-contributed skills (published for others)
│   ├── team-a/react-component-skill
│   ├── team-b/api-scaffold-skill
│   └── team-c/database-migration-skill
└── Shared MCP servers
    ├── docs-server (documentation search)
    ├── jira-server (ticket integration)
    └── monitoring-server (production metrics)
```

### 5. Cross-Team Reviews

Pair developers from different teams for AI code review:

- **Monthly rotation:** One developer reviews a PR from another team
- **Focus:** Not finding bugs, but comparing practices and sharing techniques
- **Output:** Brief write-up of interesting differences or ideas

---

## Preventing Drift

### Automated Compliance Checks

```yaml
# CI check that verifies instruction file includes org standards
name: Instruction File Compliance

on:
  pull_request:
    paths:
      - '.github/copilot-instructions.md'
      - 'CLAUDE.md'
      - 'AGENTS.md'

jobs:
  check-compliance:
    runs-on: ubuntu-latest
    steps:
      - name: Verify org standards included
        run: |
          # Check for required security sections
          grep -q "parameterized statements" .github/copilot-instructions.md
          grep -q "human review" .github/copilot-instructions.md
          grep -q "validated before processing" .github/copilot-instructions.md
```

### Champion Network as Consistency Mechanism

Champions serve as the human link between teams:

- Champions share their team's innovations in bi-weekly syncs
- Common challenges get addressed with shared solutions
- Drift is caught early through regular communication

### Dashboards and Visibility

Create visibility into AI practices across teams:

| Team | Instruction File | Prompt Library | Review Process | Last Updated |
|---|---|---|---|---|
| Frontend | ✅ Complete | ✅ 25 prompts | ✅ AI checklist | 2025-07-01 |
| Backend | ✅ Complete | ⚠️ 8 prompts | ✅ AI checklist | 2025-06-15 |
| Data | ⚠️ Partial | ❌ None | ⚠️ Basic | 2025-05-01 |
| Mobile | ✅ Complete | ✅ 15 prompts | ✅ AI checklist | 2025-07-10 |
| DevOps | ❌ None | ❌ None | ❌ None | N/A |

---

## Balancing Consistency and Autonomy

### Rules for What to Centralize

Centralize standards that:
- ✅ Involve security (non-negotiable consistency)
- ✅ Affect cross-team interactions (shared services, libraries)
- ✅ Relate to compliance (audit requirements)
- ✅ Reduce duplicated effort (common prompts everyone needs)

Let teams customize:
- ✅ Language and framework-specific conventions
- ✅ Team-specific prompt libraries
- ✅ Workflow preferences (how they use AI day-to-day)
- ✅ Additional quality rules (stricter than org minimum)

Never centralize:
- ❌ Prescribing exactly how developers should prompt
- ❌ Mandating specific AI tool usage frequency
- ❌ Requiring detailed AI usage reporting
- ❌ One-size-fits-all instruction files for different tech stacks
