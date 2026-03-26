# How Copilot Reads Instructions

> The 3-layer prompt system under the hood and what it means for your instructions.

---

## The Three Layers

When you send a request to Copilot, it processes a **stacked prompt** with three layers:

```
┌─────────────────────────────────────┐
│      Layer 1: System Prompt          │
│  (GitHub's built-in behavior rules)  │
├─────────────────────────────────────┤
│      Layer 2: Workspace Context      │
│  (Your instruction files + env info) │
├─────────────────────────────────────┤
│      Layer 3: User Task Request      │
│  (Your actual question or command)   │
└─────────────────────────────────────┘
```

### Layer 1: System Prompt (You Can't Modify)

GitHub's internal rules that govern Copilot's baseline behavior:
- Communication style and safety policies
- Output format constraints
- Compliance and content filtering
- Tool usage permissions

### Layer 2: Workspace Context (Your Instruction Files)

This is where your custom instructions live. Copilot assembles this from:

1. **Organization instructions** (if set)
2. **copilot-instructions.md** (always loaded for repo context)
3. **Matching .instructions.md files** (loaded when applyTo matches current file)
4. **Attached prompt files** (loaded when manually attached)
5. **AGENTS.md** (loaded by Coding Agent)
6. **Open files and workspace info** (IDE-reported context)

### Layer 3: User Request (Your Prompt)

Your actual question, command, or task description. This has the **highest weight** — it's the most recent and specific context.

---

## What Gets Loaded When

| Copilot Feature | What's Loaded |
|-----------------|---------------|
| **Copilot Chat** (VS Code) | System + copilot-instructions.md + matching .instructions.md + open files + your prompt |
| **Copilot Coding Agent** | System + copilot-instructions.md + matching .instructions.md + AGENTS.md + task description |
| **Copilot Code Review** | System + copilot-instructions.md + matching .instructions.md + PR diff |
| **Inline Completions** | System + open files + cursor context (instruction files have **limited** impact here) |

💡 **Key insight**: Instruction files have the strongest effect on **Chat** and **Coding Agent**. They have less impact on inline code completions (the autocomplete suggestions as you type).

---

## The Attention Problem

AI models use a mechanism called **attention** to decide which parts of their input to focus on. Key facts:

### Primacy and Recency Bias

```
[Start of context]  ← HIGH attention (primacy)
[Middle of context]  ← LOWER attention (the "lost middle" effect)
[End of context]    ← HIGH attention (recency)
```

**Implication**: The first and last sections of your instruction file get the most attention. The middle can get overlooked in long files.

### Token Competition

Every token of instruction competes with tokens of:
- The code being worked on
- File contents Copilot reads
- The conversation history
- Tool output and intermediate results

**Implication**: Longer instructions = less room for actual code context = potentially worse results.

### Relevance Filtering

Copilot doesn't mechanically execute every instruction. It **weighs relevance** — instructions that match the current task get more weight than generic rules.

```
Task: "Create a React component"

HIGH relevance: "Use functional components with TypeScript props interfaces"
LOW relevance:  "Database migrations must use nullable columns"
```

**Implication**: Instructions that are always relevant (conventions, boundaries) work better than highly specific rules that rarely apply.

---

## What Copilot Does With Instructions

### It Follows (Most of the Time)

```markdown
## Do Not
- Never use console.log in production code
```

If you ask Copilot to generate code, it will typically use a logger instead of console.log. Compliance rate: **~85-95%** for clear, specific instructions.

### It Weighs Against Other Context

```markdown
## Conventions
- Always use named exports
```

If the file Copilot is editing already uses default exports everywhere, there's a tension. Copilot may follow the existing pattern instead of the instruction. Explicit overrides help:

```markdown
## Conventions
- Always use named exports (even if existing code uses default exports — we are migrating)
```

### It Cannot Guarantee

Instructions are **strong suggestions**, not hard constraints. Copilot is a probabilistic model — it can still:
- Forget a rule in long contexts
- Prioritize code consistency over instruction compliance
- Miss an instruction when the context window is full
- Interpret ambiguous instructions differently than intended

**Mitigation**: Include validation steps so the AI can self-correct.

---

## Maximizing Instruction Effectiveness

### Principle 1: Specificity Wins

```markdown
# ❌ Vague
Write clean code.

# ✅ Specific
Use named exports. Validate input with Zod. Return proper HTTP status codes.
```

### Principle 2: Imperative Voice

```markdown
# ❌ Passive
It would be preferred if TypeScript strict mode were used.

# ✅ Imperative
Use TypeScript strict mode. Never use `any`.
```

### Principle 3: Include Verification

```markdown
# ❌ Rules only
Use TypeScript strict mode.

# ✅ Rules + verification
Use TypeScript strict mode.
After making changes, run `npm run build` to verify there are no type errors.
```

### Principle 4: Conciseness

```markdown
# ❌ Wordy (wastes tokens)
When you are writing code for this project, please make sure to always
use the TypeScript programming language with strict mode enabled, as this
is a requirement of our development team and helps catch bugs early.

# ✅ Concise (preserves tokens)
Use TypeScript strict mode; never use `any`.
```

---

## Debugging: Is Copilot Reading My Instructions?

### Check 1: References Panel

In VS Code Copilot Chat, expand the **References** at the top of any response. If `copilot-instructions.md` is listed — it's loaded.

### Check 2: Direct Ask

Ask Copilot: "What conventions should I follow in this project?"

If it quotes your instructions back to you, they're being read.

### Check 3: Behavioral Test

Ask Copilot to do something your instructions specifically address. Does the output comply?

### Check 4: Setting Verification

In VS Code: Settings → search "instruction file" → ensure "Code Generation: Use Instruction Files" is checked.

---

*Next: [Repository-Wide Instructions](../02-file-types/repository-wide-instructions.md) →*
