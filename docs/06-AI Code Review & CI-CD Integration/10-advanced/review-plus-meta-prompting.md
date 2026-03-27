# Review + Meta-Prompting Integration

> Connecting AI code review with the meta-prompting framework for end-to-end quality.

---

## The Full Loop

```
┌──────────────────────────────────────────────────────────┐
│                    Meta-Prompting Framework               │
│                                                          │
│  copilot-instructions.md  →  Guides AI code generation   │
│  .instructions.md         →  Path-specific rules         │
│  AGENTS.md                →  Agent capabilities          │
│  .prompt.md               →  Reusable prompts            │
└────────────────────────────┬─────────────────────────────┘
                             │ AI generates code
                             ▼
┌──────────────────────────────────────────────────────────┐
│                    Code Review Pipeline                   │
│                                                          │
│  copilot-instructions.md  →  Also guides AI review!      │
│  Static analysis          →  Automated checks            │
│  AI review                →  Pattern-based feedback       │
│  Human review             →  Architecture + logic         │
└────────────────────────────┬─────────────────────────────┘
                             │ Feedback loop
                             ▼
┌──────────────────────────────────────────────────────────┐
│                    Continuous Improvement                 │
│                                                          │
│  Review findings    →  Update instructions               │
│  Escaped bugs       →  Add new rules                     │
│  Team feedback      →  Refine prompts                    │
│  Metrics trends     →  Adjust thresholds                 │
└──────────────────────────────────────────────────────────┘
```

---

## The Key Insight

**The same `copilot-instructions.md` that guides code generation also guides code review.**

This means:
- Rules you write once apply to both generation and review
- AI generates code following your rules AND reviews code against those rules
- Feedback from reviews improves instructions → which improves generation

---

## Integration Points

### 1. Generation Instructions = Review Instructions
```markdown
# copilot-instructions.md

## Code Standards (applies to generation AND review)
- All API endpoints must validate input with Zod
- All database queries must use parameterized statements
- All async operations must have error handling

# When generating: follow these rules
# When reviewing: enforce these rules
```

### 2. Spec-Driven → Code → Review Loop
```
Spec (requirements.md)
  → AI generates code following spec
    → AI reviews code against spec
      → Human verifies spec compliance
        → Update spec if needed
```

### 3. AGENTS.md Defines Review Capabilities
```markdown
# AGENTS.md

## Code Review Agent
- Reviews PRs against copilot-instructions.md
- Checks spec compliance
- Validates test coverage
- Reports to: Human reviewer
```

### 4. Memory Bank Tracks Review Patterns
```markdown
# .memory-bank/review-patterns.md

## Known Issues
- AI frequently suggests unnecessary null checks for validated input
- AI misses race conditions in WebSocket handlers
- False positive rate is high for CSS-in-JS files

## Tuning History
- 2025-01: Added "ignore CSS-in-JS" rule (FP rate dropped 15%)
- 2025-02: Added WebSocket-specific review rules
```

---

## Key Takeaways

1. **Instructions serve double duty** — Same file guides generation AND review
2. **Feedback loop is automatic** — Review findings improve generation rules
3. **Spec is the source of truth** — Generation, review, and testing all reference it
4. **Memory bank captures patterns** — Cross-session learning for review tuning
5. **This is the meta-prompting advantage** — Holistic framework, not isolated tools

---

## Next Steps

- 🔗 [Templates](../11-practical/templates.md) — Ready-to-use configurations
- 🔗 [FAQ](../11-practical/faq.md) — Common questions answered
