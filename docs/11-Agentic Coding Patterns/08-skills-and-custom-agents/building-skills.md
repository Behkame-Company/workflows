# Building Custom Skills

> Step-by-step guide to creating agent skills — identify repeated tasks, codify the procedure, and teach your AI agent to perform them consistently.

---

## The Skill Building Process

```
┌──────────────────────────────────────────────────────────────┐
│              Skill Building Process                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   1. IDENTIFY       What task do you repeat?                 │
│       │                                                      │
│       ▼                                                      │
│   2. DOCUMENT       Write the procedure step-by-step         │
│       │                                                      │
│       ▼                                                      │
│   3. AUTOMATE       Add scripts for repeatable actions       │
│       │                                                      │
│       ▼                                                      │
│   4. REFERENCE      Add templates and examples               │
│       │                                                      │
│       ▼                                                      │
│   5. TEST           Run with agent, observe behavior         │
│       │                                                      │
│       ▼                                                      │
│   6. ITERATE        Refine based on agent performance        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Step 1: Identify the Task

Good skill candidates are tasks that:

- You do **repeatedly** (weekly or more often)
- Follow a **consistent procedure** each time
- Require **project-specific knowledge** the agent wouldn't know
- Are **error-prone** when done manually

### Examples of Skill-Worthy Tasks

| Task | Why It's a Good Skill |
|---|---|
| Deploy to production | Multi-step, high stakes, same every time |
| Database migration | Specific procedure, easy to miss steps |
| New API endpoint | Same pattern every time, boilerplate-heavy |
| Code review | Consistent checklist, easy to forget items |
| New React component | Same file structure, same patterns |
| Release notes | Same format, pulls from git history |
| Incident response | Critical to get right, stress reduces quality |

## Step 2: Document the Procedure

Write `instructions.md` as if explaining to a new team member:

```markdown
# [Task Name] Skill

## When to Use
[Exact conditions that should trigger this skill]

## Prerequisites
[What must be true before starting]

## Process

### Step 1: [Verb] [Object]
[Exact actions with commands]

### Step 2: [Verb] [Object]
[Exact actions with commands]

...

## Verification
[How to confirm the task was done correctly]

## Rules
[Hard constraints — things that must or must not happen]

## Troubleshooting
[Common issues and their fixes]
```

## Step 3: Add Automation Scripts

Scripts handle actions that are easier to run than describe:

```bash
#!/bin/bash
# scripts/validate.sh - Validate before deployment

set -euo pipefail

echo "Running pre-deployment validation..."

# Check tests pass
npm test --silent || { echo "Tests failed"; exit 1; }

# Check build succeeds
npm run build --silent || { echo "Build failed"; exit 1; }

# Check for uncommitted changes
if [ -n "$(git status --porcelain)" ]; then
    echo "Error: uncommitted changes detected"
    exit 1
fi

echo "Validation passed"
```

## Step 4: Add Reference Materials

Templates and examples help the agent produce consistent output:

```typescript
// resources/route-template.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

// Define your request schema
const RequestSchema = z.object({
  // Add fields here
});

// Define your response type
type ResponseData = {
  // Add fields here
};

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const validated = RequestSchema.parse(body);

    // TODO: Implement using service layer
    const result = {}; // Replace with actual implementation

    return NextResponse.json({ data: result });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: { message: "Validation failed", details: error.errors } },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: { message: "Internal server error" } },
      { status: 500 }
    );
  }
}
```

## Complete Skill Examples

### Example 1: Deployment Skill

```
.agents/skills/deploy/
├── instructions.md
├── scripts/
│   ├── pre-deploy-check.sh
│   └── post-deploy-verify.sh
└── resources/
    └── deploy-checklist.md
```

**instructions.md:**

```markdown
# Deployment Skill

## When to Use
When asked to deploy, release, or ship a new version.

## Prerequisites
- All CI checks pass on main branch
- No open P0/P1 bugs tagged for this release
- CHANGELOG.md is updated

## Process

### Step 1: Pre-Deployment Validation
Run: `bash .agents/skills/deploy/scripts/pre-deploy-check.sh`
This validates tests, build, and clean git state.

### Step 2: Version Bump
1. Determine version type from changes:
   - Breaking changes → major bump
   - New features → minor bump
   - Bug fixes only → patch bump
2. Update version: `npm version [major|minor|patch] --no-git-tag-version`
3. Update CHANGELOG.md with version header and date

### Step 3: Create Release Commit
```bash
git add package.json package-lock.json CHANGELOG.md
git commit -m "chore: release vX.Y.Z"
git tag vX.Y.Z
```

### Step 4: Push and Deploy
```bash
git push origin main --tags
```
The CI/CD pipeline handles the actual deployment from the tag.

### Step 5: Post-Deployment Verification
Run: `bash .agents/skills/deploy/scripts/post-deploy-verify.sh`
Verify the deployment is healthy.

## Rules
- Never deploy directly to production (always through CI/CD)
- Never deploy without running pre-deployment validation
- Always create a git tag for the release
- Always update CHANGELOG.md before releasing
```

### Example 2: Database Migration Skill

```
.agents/skills/db-migration/
├── instructions.md
└── resources/
    ├── migration-template.sql
    └── migration-checklist.md
```

**instructions.md:**

```markdown
# Database Migration Skill

## When to Use
When schema changes are needed: new tables, columns, indexes,
or data transformations.

## Process

### Step 1: Create Migration Files
```bash
# Generate timestamp-based migration name
npx knex migrate:make descriptive_name
```

### Step 2: Write UP Migration
In the generated file, write the forward migration:
- Use the template in resources/migration-template.sql as reference
- Add columns with sensible defaults for NOT NULL
- Add indexes for foreign keys and commonly queried columns
- Add comments for non-obvious schema decisions

### Step 3: Write DOWN Migration
In the same file, write the reverse migration:
- Must exactly undo the UP migration
- DROP in reverse order of CREATE
- Verify data is not lost (rename instead of drop when possible)

### Step 4: Test Migration
```bash
# Apply migration
npx knex migrate:latest

# Verify schema
npx knex migrate:status

# Rollback
npx knex migrate:rollback

# Re-apply to confirm idempotency
npx knex migrate:latest
```

### Step 5: Review Checklist
Verify against resources/migration-checklist.md:
- [ ] Migration is reversible
- [ ] Indexes on foreign keys
- [ ] No data loss scenarios
- [ ] Backward compatible with current code
- [ ] Tested with rollback and re-apply

## Rules
- Never modify a migration that has already been deployed
- Always test rollback before committing
- Large data migrations must process in batches (1000 rows)
- Add NOT NULL columns in two steps: add nullable → backfill → alter to NOT NULL
```

### Example 3: Component Creation Skill

```
.agents/skills/new-component/
├── instructions.md
└── resources/
    ├── component-template.tsx
    ├── story-template.tsx
    └── test-template.tsx
```

**instructions.md:**

```markdown
# New React Component Skill

## When to Use
When asked to create a new UI component.

## Process

### Step 1: Determine Component Location
- Shared/reusable → `src/components/ui/`
- Feature-specific → `src/features/[feature]/components/`
- Layout → `src/components/layout/`

### Step 2: Create Component File
Use resources/component-template.tsx as the base.
```
src/components/ui/ComponentName/
├── ComponentName.tsx
├── ComponentName.test.tsx
├── ComponentName.stories.tsx
└── index.ts
```

### Step 3: Implement Component
- Use TypeScript with explicit prop types
- Export prop type interface
- Use Tailwind for styling
- Support className prop for composition
- Handle loading, error, and empty states

### Step 4: Write Tests
Using resources/test-template.tsx:
- Renders without crashing
- Displays correct content
- Handles user interactions
- Handles edge cases (empty, loading, error)

### Step 5: Write Storybook Story
Using resources/story-template.tsx:
- Default story with typical props
- Variant stories for each state
- Interactive story for user actions

### Step 6: Export
Add to barrel export: `src/components/ui/index.ts`

### Step 7: Verify
```bash
npm test -- --testPathPattern=ComponentName
npm run storybook -- --smoke-test
npx tsc --noEmit
```

## Rules
- Never use inline styles (Tailwind only)
- Always export prop types
- Always include accessible labels (aria-label, role)
- Components must work without JavaScript (progressive enhancement)
```

## Skill Best Practices

```
┌──────────────────────────────────────────────────┐
│           Skill Design Principles                 │
├──────────────────────────────────────────────────┤
│                                                   │
│  DO                          DON'T                │
│  ──                          ─────                │
│  Clear, step-by-step         Vague descriptions   │
│  Exact commands              "Run the tests"      │
│  One task per skill          Kitchen-sink skills   │
│  Include examples            Abstract theory       │
│  Test with agent             Assume it works       │
│  Iterate on failures         Ship and forget       │
│                                                   │
│  Focused scope:                                   │
│  "deploy" ≠ "deploy + monitor + rollback"         │
│  Split into 3 separate skills                     │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Testing Your Skill

1. **Dry run:** Ask the agent to explain what it would do using the skill
2. **Simple case:** Run the skill on a straightforward task
3. **Edge case:** Run on an unusual scenario (empty input, large files)
4. **Error case:** Introduce a deliberate error and see if the skill handles it
5. **Team test:** Have a teammate use the skill without additional context

## Next Steps

- [Agent Skills Specification](./skills-spec.md) — the formal spec behind skills
- [Custom Agent Design](./custom-agent-design.md) — design agents that use your skills
- [Agent Composition](./agent-composition.md) — combine skills into workflows
- [Copilot Skills](../06-copilot-agent-patterns/copilot-skills.md) — Copilot-specific skill patterns
