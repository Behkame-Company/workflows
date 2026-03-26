# SDD vs Vibe Coding

> Understanding when to use each approach — and how to blend them.

---

## What Is Vibe Coding?

Coined by Andrej Karpathy, **vibe coding** is an informal, prompt-driven approach:

> "You just describe what you want in natural language, accept the AI's output, and iterate until it vibes."

```
Vibe Coding Loop:
  1. Prompt AI with rough description
  2. Review generated code
  3. Tweak prompt or manually fix
  4. Repeat until "it looks right"
```

---

## Head-to-Head Comparison

| Dimension | Vibe Coding | Spec-Driven Development |
|-----------|-------------|------------------------|
| **Planning** | Minimal or none | Structured specification first |
| **Speed to first output** | ⚡ Very fast | 🐢 Slower (spec takes time) |
| **Quality at scale** | 📉 Degrades rapidly | 📈 Improves with practice |
| **Maintainability** | ❌ Poor — no docs, no context | ✅ Specs are living documentation |
| **Team collaboration** | ❌ Hard to share "vibes" | ✅ Specs are reviewable artifacts |
| **Security** | ⚠️ Edge cases missed | ✅ Edge cases specified upfront |
| **Consistency** | 🎲 Varies by prompt quality | 📐 Standardized by spec templates |
| **Audit trail** | ❌ None | ✅ Full requirement traceability |
| **Context window resilience** | ❌ Lost on reset | ✅ Spec survives any context loss |
| **Best for** | Prototypes, learning, exploration | Production features, team projects |

---

## The Three-Month Wall

Vibe-coded projects almost universally hit a wall around 2–3 months:

```
Month 1:  "Wow, AI is incredible! We shipped 5 features in a week!"
Month 2:  "Why does every new feature break an old one?"
Month 3:  "Nobody understands this codebase. Let's rewrite."
```

**Root cause**: Without specifications, there's no shared understanding of:
- What each component should do (and not do)
- What edge cases have been handled
- What architectural decisions were made and why
- What "done" actually means for each feature

---

## The Hybrid Approach (Recommended)

The most effective teams **blend both approaches**:

```
Discovery Phase → Vibe Coding
  ├── Explore problem space
  ├── Prototype solutions quickly
  └── Validate feasibility

Production Phase → Spec-Driven Development
  ├── Formalize what worked
  ├── Write acceptance criteria
  ├── Plan architecture properly
  └── Implement with quality gates
```

### Decision Matrix

| Scenario | Use Vibe Coding | Use SDD |
|----------|----------------|---------|
| Hackathon or demo | ✅ | ❌ |
| Learning a new API | ✅ | ❌ |
| Throwaway prototype | ✅ | ❌ |
| Production feature | ❌ | ✅ |
| Multi-developer feature | ❌ | ✅ |
| Regulated/compliance project | ❌ | ✅ |
| Exploring feasibility | ✅ | ❌ |
| Bug fix with clear scope | ✅ | ❌ |
| Complex multi-component feature | ❌ | ✅ |
| Code for production system | ❌ | ✅ |

---

## Converting Vibe-Coded Projects to SDD

If you've been vibe-coding and want to transition:

### Step 1: Audit What Exists
```
For each major feature:
  - What does it actually do? (describe behavior)
  - What edge cases does it handle?
  - What is explicitly NOT supported?
```

### Step 2: Write Retroactive Specs
Create specs that document current behavior — even if it's imperfect.

### Step 3: Identify Gaps
Compare "what the spec says" vs. "what the code actually does." Gaps become tasks.

### Step 4: Adopt SDD Going Forward
All new features start with a spec. Existing features get specs when they need changes.

---

## Common Objections

### "SDD is too slow for our startup"
Start with lightweight specs — 10 minutes of writing a spec saves 2 hours of debugging. You don't need a 50-page document. A good spec can be 20 lines.

### "My project is too small for specs"
Even a personal project benefits from specs when you return to it after a week. Your future self (and AI agent) will thank you.

### "AI can figure out what I mean"
AI agents perform measurably better with explicit specs. A 2025 GitHub study showed **40% fewer revision cycles** when using spec-driven workflows.

---

*Next: [SDD vs TDD vs BDD →](sdd-vs-tdd-vs-bdd.md)*
