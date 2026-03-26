# Troubleshooting AGENTS.md

> When agents ignore, misread, or partially follow your instructions — and how to fix it.

---

## Problem 1: Agent Ignores AGENTS.md Entirely

### Symptoms
- Agent doesn't follow any conventions
- Agent uses default patterns instead of your specified stack
- Agent doesn't run your test commands

### Diagnosis

```
Step 1: Is the file named correctly?
  → Must be AGENTS.md (uppercase, not agents.md)

Step 2: Is it in the right location?
  → Repository root for single repos
  → Package root for monorepo packages

Step 3: Does your tool support AGENTS.md?
  → Check the compatibility matrix

Step 4: Is the file committed?
  → Some tools only read committed files, not unstaged changes

Step 5: Is the file too large?
  → If > 300 lines, tool may truncate or skip it
```

### Fixes

| Cause | Fix |
|-------|-----|
| Wrong filename | Rename to `AGENTS.md` (exact case) |
| Wrong location | Move to repository or package root |
| Tool doesn't support | Use tool-specific file + AGENTS.md |
| File not committed | `git add AGENTS.md && git commit` |
| File too large | Trim to under 200 lines |

---

## Problem 2: Agent Follows Some Rules, Ignores Others

### Symptoms
- Agent follows conventions but ignores boundaries
- Agent runs wrong test command
- Agent follows rules from the top of the file but not the bottom

### Diagnosis

The rules being ignored are likely:
1. **Too vague** — agent can't determine what to do
2. **Too far down** — lost in attention dropoff
3. **Contradicted by another rule** — agent picks one
4. **Overridden by user prompt** — direct instructions take priority

### Fixes

| Cause | Fix |
|-------|-----|
| Vague rule | Rewrite with specific tool/path/command |
| Low in file | Move critical rules to top or bottom (primacy/recency) |
| Contradiction | Audit for conflicting rules; resolve |
| Overridden by prompt | Expected behavior; AGENTS.md is lower priority than direct prompts |

### Example Fix

```markdown
❌ "Follow testing best practices" (ignored — too vague)
✅ "Test framework: Vitest. Run `pnpm test`. AAA pattern." (followed — specific)
```

---

## Problem 3: Agent Uses Wrong Commands

### Symptoms
- Agent runs `npm install` when you use pnpm
- Agent runs `jest` when you use vitest
- Agent runs wrong build command

### Diagnosis

```
Step 1: Is the correct command in AGENTS.md?
  → Check for typos in the Setup section

Step 2: Does package.json have conflicting scripts?
  → Agent may read package.json and find different commands

Step 3: Is the package manager explicitly stated?
  → "pnpm" vs "NEVER use npm"

Step 4: Is there a lockfile mismatch?
  → pnpm-lock.yaml exists but AGENTS.md says npm
```

### Fixes

```markdown
## Setup
- Package manager: pnpm (NEVER use npm or yarn)
- Install: `pnpm install`
- Test: `pnpm test`
- Build: `pnpm build`
```

Always include:
1. Explicit package manager name
2. "NEVER use [alternatives]"
3. Exact commands with all flags

---

## Problem 4: Agent Violates Boundaries

### Symptoms
- Agent modifies files in protected directories
- Agent commits .env files
- Agent adds console.log despite rule against it

### Diagnosis

```
Step 1: Is the boundary rule specific enough?
  → "Avoid X" is weaker than "NEVER X"

Step 2: Is it in the Boundaries section?
  → Rules scattered throughout are less effective

Step 3: Is the file referenced correctly?
  → "legacy/" vs "src/legacy/" — path matters

Step 4: Was the agent given conflicting instructions in the prompt?
  → "Add logging to debug this" → agent adds console.log
```

### Fixes

1. **Strengthen language**: "NEVER" instead of "avoid" or "try not to"
2. **Group boundaries**: All in one `## Boundaries` section
3. **Use full paths**: `src/legacy/` not just `legacy/`
4. **Anticipate conflicts**: If prompts might contradict, make boundary explicit
5. **Add alternatives**: "NEVER use console.log — use src/lib/logger instead"

---

## Problem 5: Agent Generates Wrong Architecture

### Symptoms
- Agent creates Redux store when you use Zustand
- Agent creates class components when you use functional
- Agent uses CSS modules when you use Tailwind

### Diagnosis

Agent is following its training data patterns, not your AGENTS.md.

### Fixes

1. **Be explicit about what you use**:
   ```markdown
   ## Stack
   Zustand (NEVER Redux, NEVER Context for global state)
   ```

2. **Be explicit about what you don't use**:
   ```markdown
   NOT: Redux, MobX, Recoil, Jotai, CSS Modules, styled-components
   ```

3. **Show the pattern**:
   ```markdown
   ## State Management Pattern
   ```typescript
   // src/stores/user.ts
   export const useUserStore = create<UserState>((set) => ({
     user: null,
     setUser: (user) => set({ user }),
   }));
   ```
   ```

---

## Problem 6: Agent Creates Files in Wrong Locations

### Symptoms
- Tests created alongside source files instead of tests/ directory
- Components created in wrong directory
- New files don't follow project structure

### Diagnosis

Agent doesn't know your directory structure, or the structure section is vague.

### Fixes

```markdown
## Structure
src/components/    — React components (create new components HERE)
src/lib/           — Utilities (create new utils HERE)
tests/unit/        — Unit tests (mirrors src/ structure)
tests/e2e/         — E2E tests

## Rules
- Test files: `tests/unit/[path matching src/].test.ts`
- Components: `src/components/[ComponentName]/index.tsx`
- Utilities: `src/lib/[feature].ts`
```

Explicit "create HERE" annotations help agents place new files.

---

## Problem 7: Agent Stops Mid-Task

### Symptoms
- Agent implements feature but doesn't add tests
- Agent writes code but doesn't run lint
- Agent creates file but doesn't commit

### Diagnosis

Agent may be:
1. Running out of context or tokens
2. Not instructed to complete the full cycle
3. Encountering an error and stopping

### Fixes

Add explicit completion checklist:

```markdown
## On Task Completion
1. Run `pnpm lint` and fix issues
2. Run `pnpm test` for affected files
3. Run `pnpm typecheck`
4. If all pass, commit with conventional commit message
5. If any fail, fix and rerun
```

---

## Problem 8: AGENTS.md Works in One Tool But Not Another

### Symptoms
- Copilot follows rules, Cursor doesn't
- Claude follows rules, Codex doesn't

### Diagnosis

Tools process AGENTS.md differently:
- Different priority levels
- Different merging behavior
- Different attention to file sections

### Fixes

1. **Test with each tool** your team uses
2. **Keep it simple** — the simpler the file, the more consistent across tools
3. **Add tool-specific files** for tools that need extra attention
4. **Put critical rules at the top** — universally highest attention zone

---

## Debugging Checklist

```
□ File named AGENTS.md (exact case)?
□ File at repository root?
□ File committed to git?
□ File under 200 lines?
□ No contradicting rules?
□ Boundaries use "NEVER" (not "avoid")?
□ Commands are exact (with flags)?
□ Package manager explicitly stated?
□ Stack includes version numbers?
□ "NOT used" list included?
□ Critical rules at top of file?
□ Tested with the specific AI tool?
```

---

*Next: [FAQ](faq.md) →*
