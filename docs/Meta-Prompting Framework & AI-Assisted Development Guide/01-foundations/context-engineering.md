# Context Engineering

> Managing the entire informational environment for AI agents — not just the prompt.

---

## From Prompt Engineering to Context Engineering

**Prompt engineering** focuses on crafting a single, well-written instruction. **Context engineering** manages the full stack of information an AI agent needs to operate effectively:

```
┌─────────────────────────────────────┐
│         System Instructions         │  ← Role, tone, boundaries
├─────────────────────────────────────┤
│      Repository Instructions        │  ← copilot-instructions.md
├─────────────────────────────────────┤
│     Path-Specific Instructions      │  ← .instructions.md files
├─────────────────────────────────────┤
│         Memory / History            │  ← Memory bank, session state
├─────────────────────────────────────┤
│     Retrieved Knowledge (RAG)       │  ← Docs, specs, examples
├─────────────────────────────────────┤
│        Tool Definitions             │  ← Available commands, APIs
├─────────────────────────────────────┤
│      Current Task / Prompt          │  ← The user's request
└─────────────────────────────────────┘
```

Every layer competes for the AI's finite **context window** — the amount of information it can process at once.

---

## The Five Pillars of Context Engineering

### 1. System Instructions

The foundational layer that defines the agent's identity and ground rules.

**Best practices:**
- Keep concise and stable — these load on every request
- Define role, tone, format expectations, and safety constraints
- Avoid task-specific details here; use path-specific instructions for that

```markdown
# Example: System-level instruction (copilot-instructions.md)
You are assisting a full-stack development team. We use TypeScript strict mode,
React 18, and Tailwind CSS. Always write tests. Never commit secrets.
```

### 2. Short-Term and Long-Term Memory

**Short-term**: Current conversation history, recent tool outputs.
**Long-term**: Persistent knowledge across sessions (memory bank files, project decisions).

**Best practices:**
- Compress long conversations — summarize rather than accumulate raw history
- Prune stale or resolved context to prevent "context rot"
- Use structured memory bank files for critical project knowledge

⚠️ **Context Rot Warning**: As the token window fills, the AI's recall per token diminishes. More context ≠ better results past a threshold.

### 3. Retrieved Knowledge (RAG)

Dynamically fetched documentation, API references, or specifications — loaded only when relevant.

**Best practices:**
- Use prompt file references (`#file:path/to/spec.md`) to attach relevant docs
- Don't dump entire codebases — fetch targeted snippets
- Prefer attaching architecture docs over letting the AI re-discover patterns

### 4. Tool Definitions

The commands, APIs, and operations the AI can perform.

**Best practices:**
- Document build/test/lint commands in your instruction files
- Include expected outputs so the AI can verify success
- List forbidden operations explicitly

### 5. Orchestration State

The current task breakdown, progress, and intermediate results.

**Best practices:**
- For complex tasks, break into phases with explicit checkpoints
- Track what's done vs. what remains
- Use memory bank's `activeContext.md` pattern for ongoing state

---

## Context Budget Management

Every AI model has a fixed context window. Managing this budget is critical:

### The Context Budget Formula

```
Available for Task = Total Context Window
                   - System Instructions
                   - Repository Instructions
                   - Path-Specific Instructions
                   - Memory/History
                   - Retrieved Knowledge
                   - Tool Definitions
```

### Optimization Strategies

| Strategy | Impact | How |
|----------|--------|-----|
| **Concise instructions** | High | Keep copilot-instructions.md under 2 pages |
| **Scoped instructions** | High | Use path-specific files so only relevant rules load |
| **Memory compression** | Medium | Summarize completed work; remove resolved decisions |
| **Selective retrieval** | Medium | Attach only relevant files, not entire directories |
| **Ignore files** | High | Use `.gitignore`-style exclusions to skip build artifacts |

### Signals of Context Overload

- AI "forgets" instructions given early in the conversation
- Responses become generic or lose project-specific conventions
- AI re-asks questions it was already told the answer to
- Quality degrades noticeably on long sessions

**Recovery**: Start a new session, update the memory bank, load fresh context.

---

## Context Layering for Teams

For team standardization, context should be layered with clear ownership:

| Layer | Owner | File | Scope |
|-------|-------|------|-------|
| Organization defaults | Platform team | Organization custom instructions | All repos |
| Repository rules | Tech lead | `copilot-instructions.md` | Whole repo |
| Domain rules | Domain owner | `.instructions.md` files | Specific paths |
| Task prompts | Individual dev | `.prompt.md` files | One-off tasks |
| Memory | Team/auto | `memory-bank/` files | Cross-session |

This creates a **clear chain of precedence**: more specific instructions override broader ones, without conflicting.

---

## Anti-Patterns

### ❌ Information Dumping
Loading every available document into context. The AI drowns in irrelevant information.

### ❌ Stale Context
Keeping outdated architectural decisions or resolved bugs in memory files. The AI acts on old information.

### ❌ Conflicting Layers
Organization instructions say "use CSS modules" but repository instructions say "use Tailwind." The AI gets confused.

### ❌ No Verification Loop
Providing extensive instructions but no way for the AI to validate its own output against them.

---

## Key Takeaways

- Context engineering manages the **full information stack**, not just prompts
- Every layer competes for a finite **context window** — budget carefully
- Use **scoped instructions** to load only what's relevant for each task
- **Compress and prune** memory to prevent context rot
- Layer context with **clear ownership** across team, repo, and path levels
- Always include **verification mechanisms** in your context stack

---

*Next: [Instruction Files Overview](instruction-files-overview.md) →*
