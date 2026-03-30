# Team Collaboration on Instructions

> Multi-developer workflows, ownership, and building consensus.

---

## The Challenge

Instruction files shape how **every developer's** AI assistant behaves. Changes affect the whole team. Without a collaboration model, you get:

- Conflicting edits (two people changing the same file)
- Style wars (one developer's preferences override another's)
- Orphan rules (nobody owns or reviews instructions)
- Drift (instructions fall out of sync with reality)

---

## Ownership Model

### Recommended: Layered Ownership

| Layer | Owner | Review Required |
|-------|-------|-----------------|
| Organization instructions | Platform/DevEx team | CTO/VP approval |
| copilot-instructions.md | Tech lead | Any senior dev review |
| frontend.instructions.md | Frontend lead | Frontend team review |
| backend.instructions.md | Backend lead | Backend team review |
| database.instructions.md | DBA / Data lead | Data team review |
| Prompt files | Any developer | Peer review |

### CODEOWNERS Configuration

```
# .github/CODEOWNERS
.github/copilot-instructions.md    @org/tech-leads
.github/instructions/frontend*     @org/frontend-team
.github/instructions/backend*      @org/backend-team
.github/instructions/database*     @org/data-team
.github/prompts/                    @org/developers
AGENTS.md                           @org/tech-leads
```

---

## Decision-Making Process

### For New Instructions

```
1. Developer identifies a pattern Copilot should follow
2. Opens a PR adding the instruction
3. PR description includes:
   - What the instruction does
   - Why it's needed (what goes wrong without it)
   - Evidence it works (tested with Copilot)
4. Assigned reviewer approves or suggests changes
5. Merged and takes effect immediately
```

### For Changing Existing Instructions

```
1. Developer proposes a change
2. PR includes:
   - What's changing and why
   - Impact on existing behavior
   - Test showing the new behavior works
3. Owner of the instruction file reviews
4. If convention change: team discussion first (RFC or meeting)
```

### For Removing Instructions

```
1. During quarterly audit or when drift is detected
2. PR removes or marks instruction as deprecated
3. Explanation: "Removing because [linter handles this now / no longer relevant]"
4. Reviewer confirms rule is truly obsolete
```

---

## Onboarding New Developers

### Day 1: Introduce Instruction Files

Include in onboarding:

1. **Read** `copilot-instructions.md` — understand project conventions
2. **Read** relevant path-specific instructions for their domain
3. **Try** a prompt file (e.g., `add-feature.prompt.md`) on a practice task
4. **Verify** their IDE loads instructions (check References panel)

### Template: Onboarding Prompt

Create a `.github/prompts/onboarding.prompt.md`:

```markdown
---
name: onboarding
description: New developer onboarding guide
---
# Welcome! Project Onboarding

Walk me through this project as a new developer:

1. What is this project and what does it do?
2. What's the tech stack?
3. How do I set up the dev environment?
4. What are the key coding conventions?
5. Where does the code for [frontend/backend/database] live?
6. How do I run the project, tests, and linter?
7. What should I know about the deployment process?

Reference: #file:.github/copilot-instructions.md
Reference: #file:README.md
```

---

## Conflict Resolution

### Common Conflicts

| Conflict | Example | Resolution |
|----------|---------|------------|
| Style preference | Tabs vs. spaces | Defer to existing linter config |
| Framework choice | CSS-in-JS vs. Tailwind | Team vote, then instruction update |
| Convention strictness | "Always" vs. "Prefer" | Default to "Always" — AI needs clarity |
| Scope disagreement | Repo-wide vs. path-specific | If exceptions exist, make it path-specific |

### Resolution Principle

**For AI instructions, clarity beats compromise.**

```markdown
# ❌ Compromise (ambiguous for AI)
- Prefer named exports when possible, but default exports are OK for pages

# ✅ Clear rule with explicit exception
- Use named exports for all files
# In pages.instructions.md:
- Page components use default exports for Next.js dynamic import compatibility
```

---

## Team Instruction Standards

### Standard: Every Instruction Must Be...

1. **Testable** — you can verify if Copilot follows it
2. **Owned** — someone is responsible for maintaining it
3. **Justified** — there's a reason it exists
4. **Current** — it reflects today's practices

### Standard: Instruction PRs Must Include...

```markdown
## Instruction Change Template

### What
[Description of the instruction being added/changed/removed]

### Why
[What goes wrong without this instruction / why the current one is inadequate]

### Test
[How you verified Copilot follows this instruction]

### Impact
[What changes in Copilot's behavior for the team]
```

---

## Scaling to Multiple Teams

### Shared Conventions Repository

For organizations with many repositories:

```
org-copilot-standards/
├── base-instructions.md        # Template for copilot-instructions.md
├── frontend-instructions.md    # Template for frontend teams
├── backend-instructions.md     # Template for backend teams
├── shared-prompts/
│   ├── add-feature.prompt.md
│   └── fix-bug.prompt.md
└── README.md                   # Usage guide
```

Teams copy and customize templates for their repos.

### GitHub Copilot Customization Repo

Check the official [Awesome GitHub Copilot Customizations](https://github.com/github/awesome-copilot) repo for community-sourced templates and examples.

---

## Communication

### When Instructions Change

Notify the team via:
- **PR description**: Explain the change
- **Slack/Teams message**: "Updated Copilot instructions: now requires Zod validation for all API routes"
- **Sprint review**: Mention instruction changes as part of team improvements

### Regular Sync

Add to existing ceremonies:
- **Sprint retro**: "Are our AI instructions working well? Any frustrations?"
- **Architecture review**: "Do instructions need updating for this change?"
- **Quarterly planning**: "Schedule instruction audit this quarter"
