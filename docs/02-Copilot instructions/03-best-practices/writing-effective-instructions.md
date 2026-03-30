# Writing Effective Instructions

> The craft of writing instructions that Copilot actually follows.

---

## The Effectiveness Spectrum

```
Ignored ←────────────────────────────────────→ Followed
  Vague            Wordy         Specific & Concise
  "write clean     "when you     "Use named exports.
   code"            write code,   Validate with Zod."
                    please..."
```

---

## The 4S Framework

Every effective instruction is **Single, Specific, Short, Surrounded**.

### 1. Single — One Rule Per Bullet

```markdown
# ❌ Multiple rules crammed together
- Use TypeScript strict mode and always validate input with Zod and never
  use console.log and make sure to add JSDoc comments

# ✅ One rule per bullet
- Use TypeScript strict mode; never use `any`
- Validate all input with Zod schemas
- Never use `console.log`; use the project logger
- Add JSDoc to all exported functions
```

**Why**: Copilot parses each bullet as a discrete instruction. Cramming multiple rules into one bullet increases the chance some are missed.

### 2. Specific — Name Tools, Libraries, and Patterns

```markdown
# ❌ Vague
- Write tests for your code
- Use proper error handling
- Follow security best practices

# ✅ Specific
- Write tests with Vitest using Arrange-Act-Assert pattern
- Wrap API handlers in try/catch; return `{ error: string, code: number }`
- Hash passwords with bcrypt (12 rounds); never store plaintext
```

**Why**: "Proper error handling" is interpretable dozens of ways. "Wrap in try/catch, return error objects" is unambiguous.

### 3. Short — Imperative Voice, No Hedging

```markdown
# ❌ Hedging and passive voice
- It would be preferred if you could try to use TypeScript strict mode
  whenever it seems appropriate for the given context
- We generally tend to lean towards using named exports in most cases,
  though there are some exceptions where default exports are acceptable

# ✅ Imperative and direct
- Use TypeScript strict mode
- Use named exports only
```

**Why**: Hedging ("try to", "whenever possible", "in most cases") signals that the rule is optional. Copilot may ignore optional-sounding rules.

### 4. Surrounded — Provide Context When Needed

```markdown
# ❌ Rule without context
- Use formatCurrency()

# ✅ Rule with essential context
- All monetary values are stored as integers (cents)
- Display money using `formatCurrency()` from `src/lib/format.ts`
```

**Why**: Copilot needs enough context to understand when and how to apply a rule.

---

## Instruction Writing Patterns

### Pattern: Command + Context

```markdown
[WHAT TO DO] — [WHY or WHERE]
```

```markdown
- Use `async/await` — never use `.then()` chains for readability
- Store dates in ISO 8601 UTC — display with `formatDate()` from `src/lib/format`
- Validate input with Zod — schemas live in `src/validators/`
```

### Pattern: Positive + Negative

```markdown
[DO this] — [DON'T do that]
```

```markdown
- Use named exports — never use default exports
- Use the project logger (`src/lib/logger`) — never use console.log
- Use Prisma Client for queries — never write raw SQL
```

### Pattern: Rule + Example

```markdown
[Rule statement]
Example: `[code snippet]`
```

```markdown
- Name components with PascalCase matching the filename
  Example: `UserProfile.tsx` exports `function UserProfile()`

- API routes return typed responses
  Example: `return NextResponse.json({ data: user })`
```

### Pattern: Conditional Rule

```markdown
When [CONDITION], [DO THIS]
```

```markdown
- When adding new API endpoints, create a Zod schema in `src/validators/`
- When modifying the database schema, run `pnpm db:migrate`
- When writing tests, use `describe` for the module and `it` for behaviors
```

---

## Tone and Voice

### ✅ Effective Voice: Technical, Direct

```markdown
- Use TypeScript strict mode
- Validate input with Zod
- Run `npm test` after changes
```

### ❌ Ineffective: Conversational, Polite

```markdown
- Hey Copilot! Could you please try to use TypeScript strict mode?
- It would be really great if you validated input with Zod 😊
- If you don't mind, run the tests when you're done
```

### ❌ Ineffective: Threatening, Conditional

```markdown
- You MUST use TypeScript or your code will be REJECTED
- NEVER EVER use any under ANY circumstances
- FAILURE to follow these rules will result in consequences
```

**Best approach**: Write like you're updating a technical wiki — neutral, authoritative, clear.

---

## Structure for Scanability

### Use Headings to Group Related Rules

```markdown
## Code Style
- TypeScript strict mode; no `any`
- Named exports only
- `async/await` over `.then()`

## Error Handling
- API routes return `{ error, code }` objects
- Client code uses Result pattern
- Never throw in API handlers

## Testing
- Vitest for unit tests
- Playwright for E2E
- Arrange-Act-Assert pattern
```

### Use "Do Not" as a Separate Section

Research shows negative constraints are parsed and followed more reliably than positive rules buried in paragraphs:

```markdown
## Do Not
- Never commit secrets or .env files
- Never modify generated files in `src/generated/`
- Never disable ESLint rules with inline comments
- Never add dependencies without team approval
```

---

## Measuring Effectiveness

### Rubric: Rate Each Instruction

| Criteria | Score |
|----------|-------|
| Is it specific? (names a tool/pattern/file) | +1 |
| Is it actionable? (Copilot can do it right now) | +1 |
| Is it testable? (you can verify compliance) | +1 |
| Is it non-obvious? (Copilot wouldn't do it naturally) | +1 |
| Is it free of hedging? (no "try to", "when possible") | +1 |

**Score 5/5**: Excellent instruction. Keep it.
**Score 3-4**: Good but could be sharper.
**Score 0-2**: Rewrite or remove. It's wasting context budget.

### Example Scoring

| Instruction | Score | Notes |
|-------------|-------|-------|
| "Write clean code" | 0/5 | Vague, not actionable, not testable |
| "Use TypeScript" | 2/5 | Somewhat specific, but Copilot may already do this |
| "Use TypeScript strict mode; never use `any`" | 4/5 | Specific, actionable, testable, direct |
| "Validate API input with Zod schemas from `src/validators/`" | 5/5 | Specific, actionable, testable, non-obvious, direct |

---

## Common Pitfalls in Instruction Writing

| Pitfall | Example | Fix |
|---------|---------|-----|
| Too vague | "Follow best practices" | Name specific practices |
| Too verbose | 3-sentence explanation per rule | One line per rule |
| Contradictory | "Use any" + "Never use any" | Pick one and remove the other |
| Aspirational | "Aim for 100% test coverage" | "All new functions need at least one test" |
| Duplicating defaults | "Write syntactically correct code" | Remove — Copilot does this anyway |
| Hedging | "Try to use TypeScript when possible" | "Use TypeScript" |
