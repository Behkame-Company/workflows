# Progressive Disclosure

> Layer-by-layer context building — start broad, dive deep only where needed.

---

## The Concept

Progressive disclosure means the agent starts with a high-level overview and progressively loads more detail only for the areas relevant to its current task.

```
Layer 0: Project structure    (20 tokens)    → I know what exists
Layer 1: Module structure     (100 tokens)   → I know how it's organized
Layer 2: File exports         (300 tokens)   → I know the public API
Layer 3: Function signatures  (500 tokens)   → I know what functions do
Layer 4: Implementation       (1,500 tokens) → I know how they work
```

Most tasks only need Layers 0-3. Only the specific code being modified needs Layer 4.

---

## Example: Fixing a Bug

```
Layer 0: Project overview
  "The project has src/, tests/, config/ directories"

Layer 1: Module structure
  "src/auth/ has jwt.ts, middleware.ts, types.ts"

Layer 2: Relevant file exports
  "jwt.ts exports: generateToken, validateToken, refreshToken"

Layer 3: Function signature
  "validateToken(token: string): Promise<TokenPayload | null>"

Layer 4: Implementation (only for the bug location)
  [Full function body — 30 lines, ~300 tokens]
```

**Total tokens**: ~500 instead of reading all 3 files (~4,500 tokens).

---

## Progressive Disclosure in Practice

### For Code Investigation
```
Phase 1: glob "src/**/*.ts" → File list (50 tokens)
Phase 2: grep "validateToken" → Where it's defined and used (150 tokens)
Phase 3: Read function signature + comments (200 tokens)
Phase 4: Read full implementation of buggy function only (300 tokens)
```

### For Architecture Understanding
```
Phase 1: Read README.md → Project purpose (500 tokens)
Phase 2: View src/ directory → Module map (100 tokens)
Phase 3: Read each module's index.ts → Public APIs (500 tokens)
Phase 4: Deep dive into the specific module being modified (varies)
```

### For Debugging
```
Phase 1: Read error message → What failed (50 tokens)
Phase 2: Read stack trace → Where it failed (200 tokens)
Phase 3: Read the failing function → Why it failed (300 tokens)
Phase 4: Read related code → Root cause (500 tokens)
```

---

## Anti-Pattern: Flat Loading

```
❌ Flat loading: Read everything at the same depth
   read_file(auth/jwt.ts)        → 1,500 tokens
   read_file(auth/middleware.ts)  → 1,200 tokens
   read_file(auth/types.ts)      → 800 tokens
   read_file(auth/utils.ts)      → 600 tokens
   Total: 4,100 tokens, mostly unnecessary

✅ Progressive: Deep only where needed
   ls src/auth/                   → 50 tokens
   grep "validateToken" src/auth/ → 150 tokens
   read jwt.ts lines 45-75       → 300 tokens (the bug)
   Total: 500 tokens, focused on the issue
```

---

## Key Takeaways

1. **Start broad, dive narrow** — Overview first, details only where needed
2. **Most tasks need Layers 0-3** — Only modify-target needs full depth
3. **8x-10x token savings** — Compared to flat loading
4. **Natural for agents** — Mirrors how humans investigate code
5. **Combine with search** — glob/grep identify where to dive deep

---

## Next Steps

- 🔗 [Context Isolation with Sub-Agents](../08-sub-agent-architectures/context-isolation-with-subagents.md) — Per-task context
- 🔗 [Minimal Viable Context](../09-best-practices/minimal-viable-context.md) — The guiding principle
