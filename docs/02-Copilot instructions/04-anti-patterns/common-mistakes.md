# Common Mistakes with Copilot Instructions

> The top 15 mistakes teams make — and how to fix each one.

---

## Mistake #1: The Empty or Missing File

**What happens**: No `.github/copilot-instructions.md` exists. Copilot uses generic defaults.

**Impact**: Copilot guesses your stack, conventions, and commands. Often wrong.

**Fix**: Create even a minimal file with just commands and stack:

```markdown
## Project
Next.js 15, TypeScript, Tailwind CSS.

## Commands
- `pnpm dev` — Start dev server
- `pnpm test` — Run tests
- `pnpm lint` — Lint code
```

---

## Mistake #2: The Novel

**What happens**: copilot-instructions.md is 3,000+ words with detailed explanations, history, and rationale for every decision.

**Impact**: Middle content gets ignored. Context budget is consumed by instructions, leaving less room for actual code.

**Fix**: Cut to 500-1,000 words. Move domain-specific content to path-specific files. Save explanations for the team wiki, not the instruction file.

---

## Mistake #3: Vague Instructions

**What happens**: Instructions like "write clean code", "follow best practices", "use proper error handling".

**Impact**: These instructions are too vague for Copilot to act on. They change nothing.

**Fix**: Replace with specific, actionable rules:

| ❌ Vague | ✅ Specific |
|---------|------------|
| Write clean code | Use named exports; never use `any` |
| Follow best practices | Validate API input with Zod |
| Use proper error handling | Catch errors in try/catch; return `{ error, code }` |
| Be secure | Never log tokens; hash passwords with bcrypt |

---

## Mistake #4: Duplicating Linter Rules

**What happens**: Instructions restate what's already in ESLint, Prettier, or TypeScript configs.

**Impact**: Wastes context budget on rules that are already machine-enforced.

**Fix**: Don't instruct Copilot about formatting, indentation, or other linter-enforced rules. Those tools catch violations automatically.

```markdown
# ❌ Redundant (ESLint/Prettier handles this)
- Use 2-space indentation
- Use single quotes for strings
- Add semicolons at end of statements
- Max line length 100 characters

# ✅ Not covered by linters (Copilot should know)
- Validate API input with Zod
- Use named exports only
- Never use `any` type
```

---

## Mistake #5: No Build/Test Commands

**What happens**: Instructions cover coding style but not how to build, test, or lint.

**Impact**: The Copilot Coding Agent **guesses** commands. It might run `npm test` when your project uses `pnpm test:ci`, or skip testing entirely.

**Fix**: Always include a Commands section. It's the single highest-impact section.

---

## Mistake #6: Contradictory Instructions

**What happens**: Repo-wide says "use named exports"; frontend.instructions.md says "use default exports for components".

**Impact**: Copilot receives both instructions and may inconsistently follow one or the other.

**Fix**: Make exceptions explicit in the path-specific file:

```markdown
# In frontend.instructions.md
Note: Page components use default exports (override to repo-wide named exports rule)
for Next.js App Router compatibility.
```

---

## Mistake #7: Instructions That Are Too Polite

**What happens**: "Please try to use TypeScript when possible 😊"

**Impact**: Hedging language ("please", "try to", "when possible") signals the rule is optional. Copilot may ignore optional-sounding rules.

**Fix**: Use imperative voice without hedging:

```markdown
# ❌ Polite hedging
- Please try to use TypeScript whenever possible
- It would be great if you could validate input

# ✅ Direct imperative
- Use TypeScript strict mode
- Validate all input with Zod
```

---

## Mistake #8: Stale Instructions

**What happens**: Instructions reference old framework versions, deleted directories, or deprecated patterns.

**Impact**: Copilot generates code using old patterns. New developers get confused by AI output.

**Fix**: Monthly quick review + quarterly full audit. See [Maintaining Instructions](../03-best-practices/maintaining-instructions.md).

---

## Mistake #9: No Negative Constraints

**What happens**: Instructions only say what to do, not what to avoid.

**Impact**: Copilot may take shortcuts or use approaches you've explicitly decided against.

**Fix**: Add a "Do Not" section:

```markdown
## Do Not
- Never use `console.log` — use the project logger
- Never modify files in `src/generated/`
- Never disable ESLint rules with inline comments
- Never add dependencies without noting in the PR description
```

---

## Mistake #10: Over-Relying on Instructions

**What happens**: Team treats instructions as the only quality gate. No linting, testing, or code review.

**Impact**: Copilot instructions are **strong suggestions**, not guarantees. The AI can still make mistakes.

**Fix**: Instructions are one layer of a defense-in-depth approach:

```
Layer 1: Copilot instructions (guidance)
Layer 2: Linter configuration (automated enforcement)
Layer 3: Type system (compile-time safety)
Layer 4: Automated tests (behavioral verification)
Layer 5: Code review (human judgment)
```

---

## Mistake #11: Putting Secrets in Instructions

**What happens**: API keys, database URLs, or credentials in instruction files.

**Impact**: Instruction files are checked into git and visible to all repo collaborators.

**Fix**: Never include actual secrets. Reference environment variables instead:

```markdown
# ❌ Secret in instructions
Database URL: postgresql://user:password@prod.example.com/mydb

# ✅ Reference to env
Database config is in .env (not committed). See .env.example for required variables.
```

---

## Mistake #12: Path-Specific Files with Wrong Globs

**What happens**: `applyTo: "src/**"` when you meant `applyTo: "src/components/**/*.tsx"`

**Impact**: Instructions load for files they shouldn't apply to, or don't load for files they should.

**Fix**: Test globs with file search:

```bash
find . -path "./src/components/**/*.tsx" -type f | head -10
```

---

## Mistake #13: Too Many Instruction Files

**What happens**: 20+ instruction files, one per tiny concern.

**Impact**: Multiple files load simultaneously, exceeding context budget. Maintenance becomes impossible.

**Fix**: Aim for 3-8 path-specific files max. Consolidate related rules.

---

## Mistake #14: Ignoring Copilot Code Review Limits

**What happens**: Instructions exceed ~4,000 characters. You expect Code Review to follow all of them.

**Impact**: Code Review silently truncates instructions beyond the limit.

**Fix**: For code review instructions, keep under 4,000 characters. If you need more, use the `excludeAgent: "code-review"` field for instructions that only apply to chat/agent.

---

## Mistake #15: Not Testing Instructions

**What happens**: Instructions are written, committed, and never verified.

**Impact**: You don't know if Copilot is actually following your rules.

**Fix**: After writing or changing instructions:

1. Open a relevant file
2. Ask Copilot to do something the instruction addresses
3. Check the References panel — is the file loaded?
4. Check the output — does it comply with the instruction?
5. If not: rewrite, reposition, or make more specific

---

## Quick Self-Assessment

Score your instruction setup:

| Question | Yes/No |
|----------|--------|
| Do you have a copilot-instructions.md? | |
| Does it include build/test commands? | |
| Is it under 1,000 words? | |
| Do you have path-specific files for different domains? | |
| Do you have a "Do Not" section? | |
| Are instructions free of hedging language? | |
| Have you tested that Copilot follows your rules? | |
| Do you review instructions at least quarterly? | |
| Is there a CODEOWNERS entry for instruction files? | |
| Do instruction PRs include test evidence? | |

**8-10 Yes**: Excellent! You're in the top tier.
**5-7 Yes**: Good foundation. Address the gaps.
**0-4 Yes**: Significant room for improvement.

---

*Next: [Instructions That Backfire](instructions-that-backfire.md) →*
