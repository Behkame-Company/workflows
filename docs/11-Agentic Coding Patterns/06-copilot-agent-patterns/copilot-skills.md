# Copilot Skills

> Building specialized agent capabilities — reusable folders of instructions, scripts, and resources that teach Copilot how to perform tasks in specific, repeatable ways.

---

## What Are Skills?

Skills are structured folders that encapsulate knowledge about how to perform a specific task. When Copilot encounters a relevant task, it loads the appropriate skill and follows its instructions.

```
┌──────────────────────────────────────────────────────────────┐
│                     What Makes a Skill                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Skill = Instructions + Scripts + Resources                 │
│                                                              │
│   ┌─────────────────┐                                        │
│   │ instructions.md  │  How to perform the task              │
│   ├─────────────────┤                                        │
│   │ scripts/         │  Automation helpers (optional)        │
│   ├─────────────────┤                                        │
│   │ resources/       │  Reference data (optional)            │
│   └─────────────────┘                                        │
│                                                              │
│   Open Standard: agentskills spec                            │
│   Works across: Copilot, Claude Code, other AI systems       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

Skills follow the **agentskills** open specification, meaning they work across different AI coding systems — not just Copilot.

## Skill Storage Locations

```
┌──────────────────────────────────────────────────────────────┐
│                    Skill Locations                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Project-Level (shared with team via git)                   │
│   ─────────────────────────────────────                      │
│   .github/skills/          Copilot-specific                  │
│   .claude/skills/          Claude Code-specific              │
│   .agents/skills/          Cross-platform                    │
│                                                              │
│   Personal (your machine only)                               │
│   ───────────────────────────                                │
│   ~/.copilot/skills/       Your Copilot skills               │
│   ~/.claude/skills/        Your Claude Code skills           │
│   ~/.agents/skills/        Your cross-platform skills        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

Project-level skills are committed to the repository and shared with the team. Personal skills live on your machine and apply to all your projects.

## Skill Folder Structure

A skill is simply a folder with at minimum an `instructions.md`:

```
my-skill/
├── instructions.md       # Required: task instructions
├── scripts/              # Optional: automation scripts
│   ├── validate.sh
│   └── setup.sh
└── resources/            # Optional: reference materials
    ├── template.ts
    └── checklist.md
```

## Creating Your First Skill

### Example: Deployment Skill

```
.github/skills/deploy/
├── instructions.md
├── scripts/
│   └── pre-deploy-check.sh
└── resources/
    └── deploy-checklist.md
```

**instructions.md:**

```markdown
# Deploy Skill

## When to Use
When asked to deploy, prepare a deployment, or release a new version.

## Process

### 1. Pre-Deployment Checks
Run the pre-deploy validation:
- All tests pass: `npm test`
- Build succeeds: `npm run build`
- No TypeScript errors: `npx tsc --noEmit`
- Lint passes: `npm run lint`

### 2. Version Bump
- Update version in `package.json`
- Update `CHANGELOG.md` with changes since last release
- Follow semver: breaking=major, feature=minor, fix=patch

### 3. Create Release
- Commit version changes: `git commit -m "chore: bump version to X.Y.Z"`
- Tag the release: `git tag vX.Y.Z`
- Push with tags: `git push origin main --tags`

### 4. Post-Deployment
- Verify the deployment at the staging URL
- Run smoke tests against staging
- Monitor error tracking for 15 minutes

## Important Rules
- Never deploy on Fridays
- Always deploy to staging first
- Rollback if error rate exceeds 1%
```

### Example: Database Migration Skill

```
.github/skills/db-migration/
├── instructions.md
└── resources/
    └── migration-template.sql
```

**instructions.md:**

```markdown
# Database Migration Skill

## When to Use
When schema changes are needed — new tables, columns, indexes, or data
transformations.

## Process

### 1. Create Migration
```bash
npx prisma migrate dev --name descriptive-name
```

### 2. Migration Rules
- Every migration must be reversible
- Include both `up` and `down` in raw SQL migrations
- Never modify an existing migration that has been deployed
- Test migration on a copy of production data

### 3. Review Checklist
- [ ] Migration is idempotent (safe to run twice)
- [ ] Indexes added for new foreign keys
- [ ] No data loss — use column addition + backfill, not column rename
- [ ] Backward compatible — old code can still read the schema
- [ ] Performance tested on production-size dataset

### 4. Data Backfill
If the migration requires data transformation:
1. Create a separate migration for the backfill
2. Process in batches (1000 rows at a time)
3. Log progress to stdout
4. Make it resumable (track last processed ID)

## Common Pitfalls
- Adding NOT NULL column without a default value
- Dropping a column that's still referenced in code
- Creating an index on a huge table without CONCURRENTLY
```

### Example: Code Review Skill

```
.github/skills/code-review/
├── instructions.md
└── resources/
    └── review-rubric.md
```

**instructions.md:**

```markdown
# Code Review Skill

## When to Use
When asked to review code, a PR, or recent changes.

## Review Process

### 1. Understand Context
- Read the PR description or commit messages
- Identify the goal of the changes
- Note which files were modified

### 2. Check for Critical Issues (Priority 1)
- Security vulnerabilities (injection, auth bypass, data exposure)
- Data loss risks (destructive operations without safeguards)
- Race conditions or concurrency bugs
- Breaking changes to public APIs

### 3. Check for Logic Issues (Priority 2)
- Off-by-one errors
- Missing error handling
- Unhandled edge cases (null, empty, boundary values)
- Resource leaks (unclosed connections, listeners)

### 4. Check for Quality Issues (Priority 3)
- Missing tests for new functionality
- Test coverage for error paths
- Naming clarity
- Function length and complexity

### 5. Output Format
Structure review as:
```
## Critical Issues (must fix)
- [file:line] Description

## Suggestions (should fix)
- [file:line] Description

## Nits (optional)
- [file:line] Description

## What Looks Good
- Brief positive notes
```

## Rules
- Never comment on style/formatting (that is the linter's job)
- Focus on issues that matter — bugs, security, logic errors
- Suggest fixes, don't just point out problems
- Be specific about line numbers and files
```

## How Agents Discover Skills

Agents automatically scan skill directories when processing a request. The discovery flow:

```
┌────────────────────────────────────────────────┐
│           Skill Discovery Flow                  │
├────────────────────────────────────────────────┤
│                                                 │
│  User Request ──▶ Agent receives task           │
│                      │                          │
│                      ▼                          │
│              Scan skill folders                  │
│              ├── .github/skills/                 │
│              ├── .agents/skills/                 │
│              └── ~/.copilot/skills/              │
│                      │                          │
│                      ▼                          │
│              Match relevant skills              │
│              (by name, instructions content)     │
│                      │                          │
│                      ▼                          │
│              Load matched skills                │
│              into agent context                  │
│                      │                          │
│                      ▼                          │
│              Agent follows skill                │
│              instructions for task              │
│                                                 │
└────────────────────────────────────────────────┘
```

## Resources and Community

### Official Resources

- **agentskills spec**: The open standard defining skill structure — [agentskills/agentskills](https://github.com/agentskills/agentskills)
- **anthropics/skills**: Community skill repository — [anthropics/skills](https://github.com/anthropics/skills)
- **github/awesome-copilot**: Curated Copilot resources — [github/awesome-copilot](https://github.com/github/awesome-copilot)

### Skill Design Best Practices

| Principle | Description |
|---|---|
| **Focused scope** | One task per skill — "deploy" not "deploy + monitor + rollback" |
| **Clear trigger** | Describe when the skill should activate |
| **Step-by-step** | Number steps in execution order |
| **Include examples** | Show expected input/output |
| **Specify commands** | Exact commands, not vague guidance |
| **Test coverage** | Test the skill with multiple scenarios |

## Building a Skill Library

Over time, build a library of skills that encode your team's knowledge:

```
.github/skills/
├── deploy/                 # Deployment workflow
├── db-migration/           # Database schema changes
├── code-review/            # Code review process
├── new-endpoint/           # API endpoint creation
├── new-component/          # React component creation
├── incident-response/      # Production incident playbook
├── onboarding/             # New developer setup
└── release-notes/          # Changelog generation
```

Each skill captures institutional knowledge that would otherwise live only in team members' heads.

## Next Steps

- [Agent Skills Specification](../08-skills-and-custom-agents/skills-spec.md) — deep dive into the agentskills spec
- [Building Custom Skills](../08-skills-and-custom-agents/building-skills.md) — step-by-step skill creation guide
- [Custom Instructions](./custom-instructions.md) — the instruction files that complement skills
- [Agent Composition](../08-skills-and-custom-agents/agent-composition.md) — combining skills into complex workflows
