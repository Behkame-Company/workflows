# Advanced Patterns

> Recursive refinement, multi-agent orchestration, and sophisticated meta-prompting techniques.

---

## Pattern 1: Recursive Prompt Refinement

Instead of writing perfect instructions once, use the AI to iteratively improve its own instruction files.

### How It Works

```
Step 1: Write initial copilot-instructions.md
Step 2: Use AI to complete a task
Step 3: Evaluate the output quality
Step 4: Ask the AI to identify what instructions would have prevented mistakes
Step 5: Update instructions based on findings
Step 6: Repeat
```

### In Practice

After the AI produces suboptimal output, prompt:

```
Review the code you just generated and compare it against our conventions.
What specific instructions in .github/copilot-instructions.md would have
prevented the issues you see? Suggest concrete additions.
```

This creates a **self-improving instruction set** — each failure makes the framework stronger.

### Cadence

- **Per-session**: Note surprising AI behaviors
- **Weekly**: Review accumulated notes, update instructions
- **Monthly**: Ask the AI to audit its own instruction files for gaps

---

## Pattern 2: Layered Specialization with Inheritance

Create a hierarchy where general rules cascade down and specialized rules override:

```
copilot-instructions.md          "Use TypeScript strict"
    ↓ inherits
frontend.instructions.md         "Use React + Tailwind"
    ↓ inherits
dashboard.instructions.md        "Use Recharts for graphs, match design system v3"
```

### Implementation with applyTo Scoping

```markdown
# .github/instructions/dashboard.instructions.md
---
applyTo: "src/features/dashboard/**/*"
---

# Dashboard Module Rules

## Additional to global and frontend rules:
- All charts use Recharts library (not D3 directly)
- Dashboard widgets must implement the `Widget` interface from `src/types/widget.ts`
- Widget data fetching uses the `useWidgetData` hook from `src/hooks/`
- Match the design system v3 spacing: 8px grid
- Loading states must show skeleton components, not spinners
```

### When to Use

- **Monorepos** with distinct subprojects
- **Large codebases** with specialized domains (auth, billing, reporting)
- **Gradual migration** where new code follows different rules than legacy code

---

## Pattern 3: Contrastive Instructions

Tell the AI what to do **and** what not to do, with examples of both:

```markdown
## Error Handling

### ✅ Correct Pattern
```typescript
try {
  const result = await userService.create(data);
  return NextResponse.json(result, { status: 201 });
} catch (error) {
  logger.error('User creation failed', { error, data: sanitize(data) });
  if (error instanceof ValidationError) {
    return NextResponse.json({ error: error.message }, { status: 400 });
  }
  return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
}
```

### ❌ Wrong Pattern
```typescript
// DON'T: generic catch, console.log, no status differentiation
try {
  const result = await userService.create(data);
  return NextResponse.json(result);
} catch (e) {
  console.log(e);
  return NextResponse.json({ error: 'Something went wrong' });
}
```
```

### Why This Works

AI models respond strongly to contrastive examples. Showing both the desired and undesired pattern creates a stronger signal than either alone.

---

## Pattern 4: Decision Trees in Instructions

For complex scenarios, use conditional logic:

```markdown
## When Creating New Files

**If it's a React component:**
1. Create in `src/components/ComponentName/`
2. Include `index.tsx` + `ComponentName.test.tsx`
3. Export from `src/components/index.ts`

**If it's an API route:**
1. Create in `src/app/api/resource-name/route.ts`
2. Include Zod schema in `src/lib/validators/`
3. Add to the OpenAPI spec

**If it's a utility function:**
1. Add to existing file in `src/lib/` if related to current utilities
2. Create new file only if it's a distinct domain
3. Always add unit tests

**If it's a database change:**
1. Modify `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name descriptive_name`
3. Update seed data if needed
4. Never delete columns without a deprecation migration first
```

---

## Pattern 5: Progressive Disclosure

Don't load everything at once. Layer context based on task complexity:

### Level 1: Always Loaded (copilot-instructions.md)
Core conventions, commands, and safety rules.

### Level 2: Domain-Triggered (.instructions.md)
Loaded only when working on matching files — frontend rules when editing `.tsx`, etc.

### Level 3: On-Demand (.prompt.md)
Complex workflow templates attached by the developer when needed.

### Level 4: Deep Context (Memory Bank)
Full project context attached only for strategic or cross-cutting tasks.

```
Simple task:     Level 1 only                        (~500 tokens)
Domain task:     Level 1 + Level 2                   (~1,000 tokens)
Feature work:    Level 1 + Level 2 + Level 3         (~2,000 tokens)
Architecture:    Level 1 + Level 2 + Level 3 + Level 4  (~4,000 tokens)
```

This preserves context budget for the actual work.

---

## Pattern 6: Checkpoint-Based Workflows

For complex multi-step tasks, define explicit checkpoints:

```markdown
# .github/prompts/complex-feature.prompt.md

# Complex Feature Implementation

## Checkpoint 1: Planning
- [ ] Read memory-bank/activeContext.md
- [ ] List all files that will change
- [ ] Identify dependencies and risks
- [ ] Get approval before proceeding
STOP — Wait for review before continuing.

## Checkpoint 2: Types & Interfaces
- [ ] Define all new TypeScript types
- [ ] Run `npm run build` to verify type correctness
STOP — Verify types match the plan.

## Checkpoint 3: Backend
- [ ] Implement API routes with validation
- [ ] Write API tests
- [ ] Run `npm run test` — must pass
STOP — Verify API works before building UI.

## Checkpoint 4: Frontend
- [ ] Implement UI components
- [ ] Write component tests
- [ ] Run full test suite
STOP — Verify everything works together.

## Checkpoint 5: Integration
- [ ] Run `npm run test:e2e`
- [ ] Verify against acceptance criteria
- [ ] Update memory-bank/activeContext.md
DONE
```

### Why Checkpoints Matter

- Prevent the AI from going too far down a wrong path
- Create natural review points for the developer
- Make it possible to roll back to a known-good state
- Reduce the cost of mistakes (caught earlier)

---

## Pattern 7: Meta-Prompt Audit Loop

Periodically have the AI evaluate and improve the entire framework:

### Audit Prompt

```
Analyze our current meta-prompting framework:

1. Read all files in .github/ (copilot-instructions.md, instructions/, prompts/)
2. Read memory-bank/ files
3. Read AGENTS.md

Then answer:
- Are there contradictions between any instruction files?
- Are any instructions too vague to be actionable?
- What common development tasks have no corresponding prompt file?
- Are memory bank files current or stale?
- What instructions are missing based on common patterns in our codebase?

Provide specific, actionable recommendations.
```

Schedule this monthly to prevent framework decay.

---

## Pattern 8: Team-Specific Instruction Variants

For larger teams with specialized roles:

```
.github/
├── copilot-instructions.md           # Everyone
├── instructions/
│   ├── frontend.instructions.md      # Frontend team
│   ├── api.instructions.md           # Backend team
│   ├── devops.instructions.md        # DevOps/infrastructure
│   └── junior-dev.instructions.md    # Extra guardrails for junior developers
└── prompts/
    ├── senior-refactor.prompt.md     # Senior devs: refactoring workflow
    └── onboarding-task.prompt.md     # New team members: guided workflow
```

The `applyTo` patterns can match not just file paths but entire directory structures, allowing you to tailor instructions for different parts of the codebase that different team roles work on.

---

## Summary: Choosing Patterns

| Pattern | Use When |
|---------|----------|
| Recursive Refinement | Building initial instructions; ongoing improvement |
| Layered Specialization | Monorepos; large codebases with distinct domains |
| Contrastive Instructions | AI keeps making the same mistakes |
| Decision Trees | Multiple valid approaches depending on context |
| Progressive Disclosure | Context budget is tight; tasks vary in complexity |
| Checkpoint Workflows | Complex multi-step features; need review gates |
| Audit Loop | Monthly maintenance; preventing framework decay |
| Team Variants | Teams with different experience levels or roles |

---

*Next: [Spec-Driven Development Overview](../06-spec-driven-development/spec-driven-overview.md) →*
