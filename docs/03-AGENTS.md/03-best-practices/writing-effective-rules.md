# Writing Effective AGENTS.md Rules

> How to craft instructions that AI coding agents actually follow — with tested patterns and compliance data.

---

## The Rule Writing Framework

Every effective rule follows this pattern:

```
[VERB] [SPECIFIC THING] [IN/FROM/WITH SPECIFIC CONTEXT]
```

### Examples

```markdown
✅ "Use named exports from all files in src/components/"
✅ "Run `pnpm test` before committing"
✅ "Validate all API input with Zod schemas"
✅ "NEVER modify files in migrations/"

❌ "Try to keep things consistent"
❌ "Follow our coding standards"
❌ "Be careful with the database"
```

---

## The Compliance Spectrum

Not all instructions are created equal. Agent compliance depends on **specificity**, **clarity**, and **testability**:

| Rule Type | Compliance | Example |
|-----------|-----------|---------|
| Exact commands | ~95% | `Run: pnpm test --run` |
| Hard boundaries | ~90% | `NEVER modify /legacy/` |
| Named patterns | ~85% | `Use Zod for validation` |
| Style rules | ~75% | `Named exports only` |
| Architecture guidance | ~65% | `Server Components by default` |
| Soft preferences | ~50% | `Prefer early returns` |
| Vague platitudes | ~5% | `Write clean code` |

**Key insight**: If you can't test whether a rule was followed, the agent can't follow it either.

---

## 10 Patterns for High-Compliance Rules

### Pattern 1: Name the Tool

```markdown
❌ "Use a validation library"
✅ "Validate input with Zod. Never use Yup or manual validation."
```

Agents know thousands of validation libraries. Name the one you want.

### Pattern 2: Show the Alternative You DON'T Want

```markdown
❌ "Use proper error handling"
✅ "Throw AppError from src/lib/errors. Never throw raw Error or string."
```

Closing the wrong door is as important as opening the right one.

### Pattern 3: Include File Paths

```markdown
❌ "Put shared types in the right place"
✅ "Shared TypeScript types go in src/types/. Per-feature types stay in their feature directory."
```

Agents need explicit paths, not implied conventions.

### Pattern 4: Version-Pin When It Matters

```markdown
❌ "We use React"
✅ "React 18 with Server Components (Next.js 15 App Router). Never use pages/ directory."
```

The difference between React 17 and 18 is significant for code generation.

### Pattern 5: Use Triplets (DO, DON'T, WHY)

```markdown
✅ ## Database
   - DO: Use Prisma client for all queries
   - DON'T: Write raw SQL or use query builders
   - WHY: Type safety and migration tracking
```

The "why" helps agents make correct judgment calls in edge cases.

### Pattern 6: Quantify When Possible

```markdown
❌ "Keep functions short"
✅ "Maximum 30 lines per function. Maximum 3 parameters."
```

Numbers are unambiguous.

### Pattern 7: Scope the Rule

```markdown
❌ "Always add tests"
✅ "Add unit tests for all new functions in src/lib/ and src/server/.
    Component tests for new components in src/components/.
    No tests required for types, configs, or migration files."
```

Scoped rules prevent over-application.

### Pattern 8: Define the Pattern With Code

```markdown
✅ ## Hook Pattern
   ```typescript
   export function useFeature() {
     const [state, setState] = useState<FeatureState>(initialState);

     const action = useCallback(async () => {
       // implementation
     }, [dependency]);

     return { state, action } as const;
   }
   ```
```

One example replaces paragraphs of description.

### Pattern 9: Create Tiers (Must / Should / May)

```markdown
## Rules

### Must (CI enforced)
- TypeScript strict mode
- All tests pass
- No lint errors

### Should (PR review)
- Component tests for user-facing changes
- Explicit return types on exported functions

### May (team preference)
- Early returns for guard clauses
- Ternary for simple conditionals
```

Tiered rules help agents prioritize when rules conflict.

### Pattern 10: Address Common Agent Mistakes

Watch for what agents get wrong, then add targeted rules:

```markdown
## Common Mistakes to Avoid
- Don't import server-only modules in 'use client' components
- Don't use React.FC — use explicit props interface
- Don't create index.ts barrel files (causes circular imports)
- Don't add `@ts-ignore` — fix the type error instead
```

---

## The Feedback Loop: Improving Rules Over Time

```
1. Write initial AGENTS.md (30 lines)
         ↓
2. Use agent for real tasks
         ↓
3. Observe: What did the agent get wrong?
         ↓
4. Add targeted rule to prevent that specific error
         ↓
5. Verify: Does the agent now get it right?
         ↓
6. If yes → keep rule. If no → rewrite rule with more specificity.
         ↓
7. Repeat from step 2.
```

**This is the most effective way to build a great AGENTS.md**: start small, observe errors, add rules, and iterate.

---

## Rule Density: How Many Is Too Many?

| Category | Recommended Max |
|----------|----------------|
| Setup commands | 10-15 lines |
| Conventions | 5-10 rules |
| Boundaries | 5-10 rules |
| Code patterns | 2-3 examples |
| Testing | 5-8 rules |
| **Total AGENTS.md** | **50-100 lines** |

Beyond ~100 lines, add more through per-directory AGENTS.md files rather than growing the root file.
