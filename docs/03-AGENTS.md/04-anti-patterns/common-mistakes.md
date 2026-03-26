# Common Mistakes in AGENTS.md

> The top 12 mistakes teams make — with real examples and fixes.

---

## Mistake 1: The Empty Platitude File

### ❌ What It Looks Like

```markdown
# AGENTS.md
You are a helpful coding assistant. Write clean, well-structured code
following best practices. Make sure tests pass and code is readable.
```

### Why It Fails

This tells the agent nothing it doesn't already know. "Clean code" and "best practices" are not actionable instructions.

### ✅ Fix

Replace with specific, testable rules:
```markdown
# AGENTS.md
## Setup
- Test: `pnpm test`
- Lint: `pnpm lint`

## Conventions
- TypeScript strict mode; never use `any`
- Named exports only
- Validate API input with Zod

## Boundaries
- NEVER modify migrations/
- NEVER commit .env files
```

---

## Mistake 2: The Novel (300+ Lines)

### ❌ What It Looks Like

A 500-line file covering every possible scenario, with lengthy explanations, extensive code examples, historical context, and team philosophy.

### Why It Fails

- Agent compliance drops below 40% past 300 lines
- Critical rules get buried in noise
- High token cost reduces space for actual code
- Maintenance burden becomes unsustainable

### ✅ Fix

Trim to essential rules. Split into per-directory files for monorepos. Target 50-100 lines.

---

## Mistake 3: The README Duplicate

### ❌ What It Looks Like

```markdown
# AGENTS.md

## About This Project
MyApp is a full-stack web application built for managing...

## Getting Started
1. Clone the repository
2. Navigate to the project directory
3. ...

## Contributing
Please read our CONTRIBUTING.md guide...
```

### Why It Fails

Agents already read README.md. Duplicating it in AGENTS.md wastes tokens and adds zero value.

### ✅ Fix

AGENTS.md should contain information that README.md doesn't: machine-specific rules, boundaries, and conventions that agents need but humans don't.

---

## Mistake 4: Missing Setup Commands

### ❌ What It Looks Like

```markdown
# AGENTS.md
## Conventions
- Use TypeScript
- Follow our coding standards
- Write tests
```

No setup section. No commands.

### Why It Fails

The agent doesn't know how to:
- Install dependencies
- Run tests
- Build the project
- Start the dev server

It will guess — and guess wrong.

### ✅ Fix

Always include exact commands:
```markdown
## Setup
- Install: `pnpm install`
- Dev: `pnpm dev`
- Test: `pnpm test`
- Build: `pnpm build`
```

---

## Mistake 5: Wrong Package Manager

### ❌ What It Looks Like

```markdown
## Setup
- Install: `npm install`
- Test: `npm test`
```

But the project uses pnpm (has pnpm-lock.yaml).

### Why It Fails

Agent runs `npm install`, creating a conflicting `package-lock.json`. Tests may behave differently. CI may fail.

### ✅ Fix

Match your actual package manager. Be explicit about what NOT to use:
```markdown
## Setup
- Package manager: pnpm (NEVER use npm or yarn)
- Install: `pnpm install`
```

---

## Mistake 6: No Boundaries

### ❌ What It Looks Like

An AGENTS.md with conventions and setup but zero "NEVER" rules.

### Why It Fails

Without boundaries, agents may:
- Modify migration files
- Commit secrets
- Delete important configurations
- Refactor legacy code you're preserving
- Install unwanted dependencies

### ✅ Fix

Add 5-7 boundary rules. Start with the most dangerous actions:
```markdown
## Boundaries
- NEVER commit .env, secrets, or API keys
- NEVER modify existing migration files
- NEVER modify files in legacy/ or generated/
- NEVER delete database columns
- NEVER disable lint rules with inline comments
```

---

## Mistake 7: Soft Language for Hard Rules

### ❌ What It Looks Like

```markdown
- Please try to avoid using console.log
- It would be nice if you used TypeScript
- We prefer named exports when possible
- Consider adding tests for new features
```

### Why It Fails

Agents interpret "try to," "nice if," "prefer," and "consider" as optional suggestions. Compliance drops to ~25-40%.

### ✅ Fix

Use imperative language:
```markdown
- NEVER use console.log (use src/lib/logger)
- Use TypeScript for all source files
- Use named exports only
- All new functions in src/ must have tests
```

---

## Mistake 8: Stale References

### ❌ What It Looks Like

```markdown
## Stack
React 16, Webpack 4, Redux, Jest.

## Structure
src/pages/         — Route pages
src/reducers/      — Redux reducers
src/store/         — Redux store
```

But the project migrated to React 18, Vite, Zustand, and Vitest 6 months ago.

### Why It Fails

Agent generates code for the wrong framework version, uses deprecated APIs, and creates files in non-existent directories.

### ✅ Fix

Schedule quarterly AGENTS.md audits. Update when you update dependencies.

---

## Mistake 9: Instructions That Agents Can't Verify

### ❌ What It Looks Like

```markdown
- Ensure the code is performant
- Make sure the UX is intuitive
- Keep the architecture scalable
```

### Why It Fails

Agents can't measure "performant," "intuitive," or "scalable." These are human judgments.

### ✅ Fix

Convert to measurable rules:
```markdown
- No N+1 queries — use eager loading for related data
- All interactive elements must have aria labels
- Maximum 3 levels of component nesting
```

---

## Mistake 10: Tool-Specific Instructions in AGENTS.md

### ❌ What It Looks Like

```markdown
# AGENTS.md
Hey Copilot, when you generate code, make sure to use TypeScript.
Claude, please remember to check types before committing.
```

### Why It Fails

AGENTS.md is tool-agnostic. Addressing specific tools:
- Confuses other tools that read the file
- Breaks if tool names change
- Reduces portability

### ✅ Fix

Write universal instructions. Put tool-specific content in tool-specific files:
```markdown
# AGENTS.md (universal)
- Use TypeScript strict mode
- Run type check before committing: `pnpm typecheck`
```

---

## Mistake 11: Including External Links

### ❌ What It Looks Like

```markdown
## Style Guide
For coding standards, see our wiki: https://wiki.company.com/style-guide
For API documentation, visit: https://docs.company.com/api
```

### Why It Fails

AI coding agents cannot browse URLs. The instructions point to information the agent will never access.

### ✅ Fix

Include the critical rules inline:
```markdown
## Style Guide
- Naming: camelCase for functions, PascalCase for components
- Max line length: 100 characters
- Imports: external → internal → relative
```

---

## Mistake 12: Contradictory Rules

### ❌ What It Looks Like

```markdown
## Conventions
- Use TypeScript strict mode
- Type assertions are acceptable when working with third-party libraries

## Boundaries
- NEVER use type assertions
```

### Why It Fails

The agent receives contradictory instructions and must choose. Its choice may not match your intent.

### ✅ Fix

Resolve conflicts before committing:
```markdown
## Conventions
- TypeScript strict mode
- NEVER use `as` type assertions (use type guards instead)
- Exception: `as unknown as Type` permitted for test mocks only
```

---

## Quick Reference: Mistake → Fix

| # | Mistake | Fix |
|---|---------|-----|
| 1 | Empty platitudes | Specific, testable rules |
| 2 | Too long (300+ lines) | Trim to 50-100 lines; split for monorepo |
| 3 | README duplicate | Agent-specific content only |
| 4 | Missing commands | Add exact setup/test/build commands |
| 5 | Wrong package manager | Match actual PM; state "NEVER use X" |
| 6 | No boundaries | Add 5-7 "NEVER" rules |
| 7 | Soft language | Imperative voice; "NEVER" not "avoid" |
| 8 | Stale references | Quarterly audits |
| 9 | Unverifiable rules | Measurable, concrete alternatives |
| 10 | Tool-specific content | Universal rules; tool files for specifics |
| 11 | External links | Inline the critical rules |
| 12 | Contradictory rules | Review and resolve before committing |

---

*Next: [Instructions That Backfire](instructions-that-backfire.md) →*
