# Structuring & Organizing Instructions

> File layout, sections, and scoping strategy for projects of all sizes.

---

## Organizing Principle: Context Efficiency

The goal is to give Copilot the **right instructions at the right time** — not all instructions all the time.

```
BAD:  One giant file with everything → context overload
BAD:  20 tiny files → overhead, hard to maintain
GOOD: 1 repo-wide + 3-5 path-specific + 2-4 prompt files
```

---

## Size Recommendations by Project Scale

### Small Project (1-5 developers)

```
.github/
├── copilot-instructions.md     # 300-500 words — everything in one file
└── prompts/
    └── add-feature.prompt.md   # Optional workflow prompt
```

**Rationale**: Small projects don't need scoping. One concise file covers everything.

### Medium Project (5-20 developers)

```
.github/
├── copilot-instructions.md     # 500-800 words — universal rules
├── instructions/
│   ├── frontend.instructions.md
│   ├── backend.instructions.md
│   └── testing.instructions.md
└── prompts/
    ├── add-feature.prompt.md
    ├── fix-bug.prompt.md
    └── generate-tests.prompt.md
```

**Rationale**: Separate frontend/backend domains. Shared prompt workflows.

### Large Project / Monorepo (20+ developers)

```
.github/
├── copilot-instructions.md     # 300-500 words — minimal universal rules
├── instructions/
│   ├── frontend-react.instructions.md
│   ├── frontend-styles.instructions.md
│   ├── api-routes.instructions.md
│   ├── api-middleware.instructions.md
│   ├── database.instructions.md
│   ├── auth.instructions.md
│   ├── testing-unit.instructions.md
│   ├── testing-e2e.instructions.md
│   └── infrastructure.instructions.md
└── prompts/
    ├── add-feature.prompt.md
    ├── fix-bug.prompt.md
    ├── generate-tests.prompt.md
    ├── create-migration.prompt.md
    ├── code-review-prep.prompt.md
    └── onboarding.prompt.md
```

**Rationale**: Highly scoped instructions by domain and concern. Each file stays focused and small.

---

## Section Organization Within Files

### copilot-instructions.md — Recommended Order

```markdown
1. ## Project         ← Identity (what is this, what stack)
2. ## Commands        ← How to build/test/lint (HIGHEST impact)
3. ## Structure       ← Where things live
4. ## Conventions     ← Coding rules (5-10 max)
5. ## Do Not          ← Hard boundaries
6. ## Validation      ← Self-check steps
```

### Path-Specific Files — Recommended Order

```markdown
1. (frontmatter)     ← applyTo glob
2. ## Context         ← What this domain is about (1-2 sentences)
3. ## Conventions     ← Domain-specific rules
4. ## Patterns        ← Expected code patterns (with examples)
5. ## Do Not          ← Domain-specific boundaries
```

---

## Scoping Decision Framework

### Decision: Repo-wide or path-specific?

```
Does this rule apply to ALL files in the repo?
├── YES → copilot-instructions.md
└── NO ─→ Does it apply to a specific directory or file type?
          ├── YES → .instructions.md with applyTo
          └── NO ─→ Is it a one-time workflow?
                    ├── YES → .prompt.md
                    └── NO ─→ Probably doesn't need an instruction file
```

### Decision: One file or split?

```
Do the rules share a cohesive theme?
├── YES → One file (e.g., "frontend.instructions.md")
└── NO ─→ Are there more than 15 rules?
          ├── YES → Split by concern (e.g., "frontend-components" + "frontend-styles")
          └── NO ─→ Keep in one file
```

---

## File Naming Conventions

### Instruction Files

```
[domain].instructions.md
[domain]-[concern].instructions.md
```

| ✅ Good Names | ❌ Bad Names |
|--------------|-------------|
| `frontend.instructions.md` | `my-rules.instructions.md` |
| `api-routes.instructions.md` | `stuff.instructions.md` |
| `database.instructions.md` | `important.instructions.md` |
| `testing-unit.instructions.md` | `rules-v2.instructions.md` |

### Prompt Files

```
[verb]-[noun].prompt.md
```

| ✅ Good Names | ❌ Bad Names |
|--------------|-------------|
| `add-feature.prompt.md` | `my-prompt.prompt.md` |
| `fix-bug.prompt.md` | `prompt1.prompt.md` |
| `generate-tests.prompt.md` | `help.prompt.md` |

---

## Cross-Reference Strategy

### Prompt Files Referencing Instructions

```markdown
# In add-feature.prompt.md
Follow all conventions from #file:.github/copilot-instructions.md
```

### Instructions Referencing Each Other

Keep instruction files independent. If you find yourself saying "also see X.instructions.md," the rules probably belong in one file.

### Instructions Referencing Project Files

```markdown
## Error Handling
Use the AppError class from `src/lib/errors.ts`.
Return error responses following the format in `src/types/api.ts`.
```

---

## Anti-Patterns in Organization

### ❌ The Mega File

One 3,000-word copilot-instructions.md with everything. Middle content gets ignored.

**Fix**: Extract domain-specific rules into path-specific files.

### ❌ The Micro Files

15 instruction files with 3 lines each. Too much overhead, impossible to maintain.

**Fix**: Consolidate related rules. Aim for 5-15 rules per file.

### ❌ The Duplication Problem

Same rule in copilot-instructions.md AND frontend.instructions.md.

**Fix**: Put universal rules in copilot-instructions.md only. Path-specific files contain only domain-specific additions.

### ❌ The Orphan Prompt

A .prompt.md file that references conventions not in any instruction file.

**Fix**: Either add the conventions to an instruction file or embed them in the prompt.

---

## Maintenance Checklist

When files change, verify:

- [ ] No contradictions between repo-wide and path-specific files
- [ ] No duplicated rules across files
- [ ] All `applyTo` patterns still match the current directory structure
- [ ] Prompt files reference instructions that still exist
- [ ] Total instruction content stays under context budget
