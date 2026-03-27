# Debugging Context Issues

> When the AI goes off track — how to diagnose and fix context-related problems.

---

## Common Symptoms & Diagnoses

### Symptom: Model Forgets Instructions
```
You: "Use TypeScript strict mode"
[... 20 turns later ...]
Model: *generates JavaScript with `any` types*
```

**Diagnosis**: Instructions buried in long context (lost-middle effect).
**Fix**: Re-inject critical instructions near the end of context, or compact and re-state.

---

### Symptom: Model Contradicts Itself
```
Turn 5: "We should use microservices for scalability"
Turn 15: "I'll implement this as a monolith for simplicity"
```

**Diagnosis**: Earlier decision buried too deep in context.
**Fix**: Record decisions in a persistent file (decisions.md). Re-reference before continuing.

---

### Symptom: Model Hallucinates File Names
```
Model: "Let me read src/auth/verifyUser.ts"
Reality: That file doesn't exist
```

**Diagnosis**: Model's context doesn't include actual file listing.
**Fix**: Run `glob` or `ls` before file operations. Include project structure in upfront context.

---

### Symptom: Model Repeats Work Already Done
```
Turn 10: Model implements user registration
Turn 30: Model implements user registration again
```

**Diagnosis**: Earlier implementation compacted away or lost to context rot.
**Fix**: Use a feature tracking file (feature_list.json). Update after each completion.

---

### Symptom: Model Gets Slower and Less Accurate
```
Turns 1-10: Fast, accurate responses
Turns 30-50: Slow, rambling, less focused
```

**Diagnosis**: Context window filling up; attention budget diluted.
**Fix**: Compact, clear old tool results, or start a new session.

---

## Diagnostic Toolkit

### 1. Check Context Size
```python
# Count tokens before sending
count = client.messages.count_tokens(messages=messages)
print(f"Current context: {count.input_tokens} tokens")
print(f"Window utilization: {count.input_tokens / 200000 * 100:.1f}%")
```

### 2. Audit Context Contents
Review what's actually in the context:
- How many tool results are still present?
- Are there old conversation turns about completed tasks?
- Is there redundant information?

### 3. Test Recall
Ask the model to recall specific information:
- "What tech stack are we using?"
- "What have we completed so far?"
- "What are our coding standards?"

If it can't answer these accurately, context needs cleanup.

### 4. The Nuclear Option
When context is hopelessly polluted:
1. Save current state to files (progress notes, decisions)
2. Start a completely new conversation
3. Load only the minimum viable context
4. Continue from the clean state

---

## Prevention Checklist

- [ ] System prompt has critical rules at the start
- [ ] Tool results are cleared after processing
- [ ] Compaction is configured and triggered appropriately
- [ ] Progress tracking file is updated regularly
- [ ] New conversations started for unrelated tasks
- [ ] Decisions are recorded in persistent files
- [ ] Context size is monitored

---

## Key Takeaways

1. **Most AI "hallucinations" are context problems** — Not model failures
2. **Lost-middle effect is real** — Re-inject critical rules
3. **Test recall periodically** — Catch degradation early
4. **Persistent files are insurance** — Survive compaction and resets
5. **When in doubt, start fresh** — Clean context beats polluted context

---

## Next Steps

- 🔗 [Tool-Specific Strategies](tool-specific-strategies.md) — Per-tool guidance
- 🔗 [Context Stuffing](../10-anti-patterns/context-stuffing.md) — The #1 anti-pattern
