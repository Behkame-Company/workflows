# Troubleshooting

> Common context window problems and how to fix them.

---

## Model Forgets Instructions

**Symptom**: Model ignores rules from the system prompt.
**Cause**: Instructions lost in the middle of long context.
**Fixes**:
1. Re-inject critical rules at the end of the conversation
2. Compact older turns to bring instructions closer to recent context
3. Use XML tags to make instructions stand out: `<critical_rules>...</critical_rules>`
4. Start a new conversation for long sessions

---

## Model Generates Wrong Code Style

**Symptom**: Model uses different patterns than your project.
**Cause**: copilot-instructions.md / CLAUDE.md not specific enough, or buried in context.
**Fixes**:
1. Add explicit code examples to your instructions
2. Reference actual project files: "Follow the pattern in src/api/users.ts"
3. Close irrelevant files (IDE tools) so model sees correct patterns
4. Reduce instructions to 10-15 critical rules

---

## Session Gets Slow and Unfocused

**Symptom**: Responses get slower, longer, less relevant.
**Cause**: Context window filling up; attention diluted.
**Fixes**:
1. Check context utilization (are you over 75%?)
2. Clear old tool results that are no longer needed
3. Compact earlier conversation turns
4. Start a new conversation if too polluted

---

## Model Repeats Completed Work

**Symptom**: Model implements something that was already done.
**Cause**: No progress tracking; earlier implementation compacted away.
**Fixes**:
1. Maintain feature_list.json with completion status
2. Update progress.txt after each sub-task
3. Use git commits as progress markers
4. Reference progress file in your prompts

---

## Compaction Loses Critical Context

**Symptom**: After compaction, model makes decisions contradicting earlier ones.
**Cause**: Default compaction dropped an important detail.
**Fixes**:
1. Use custom compaction instructions that prioritize decisions
2. Record decisions in persistent files (decisions.md) before compaction
3. Re-inject critical context after compaction
4. Use `pause_after_compaction` to add context after summary

---

## Model Hallucinates File Names

**Symptom**: Model references files that don't exist.
**Cause**: Model guessing from patterns instead of checking actual filesystem.
**Fixes**:
1. Prompt: "Always use glob/ls to verify files exist before referencing them"
2. Include project file structure in upfront context
3. Use targeted reads with verified file paths

---

## Context Overflow Error

**Symptom**: API returns "context window exceeded" error.
**Cause**: Total tokens (input + output) exceed window limit.
**Fixes**:
1. Enable compaction with trigger at 75% of window
2. Use token counting API to check before sending
3. Reduce system prompt size
4. Clear old tool results and conversation turns
5. Use a model with a larger window

---

## Key Takeaways

1. **Most issues are context problems** — Not model intelligence problems
2. **Compaction solves 80% of issues** — Enable it and tune it
3. **Persistent files are insurance** — Decisions and progress survive resets
4. **New conversations are free** — Don't cling to polluted context
5. **Monitor utilization** — Catch problems before they manifest

---

## Next Steps

- 🔗 [FAQ](faq.md) — Quick answers
- 🔗 [Resources](resources.md) — Further reading
