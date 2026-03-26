# Frequently Asked Questions

> 25 answers to the most common questions about Copilot instruction files.

---

## Setup & Basics

### Q1: Where does copilot-instructions.md go?

`.github/copilot-instructions.md` — in the `.github/` directory at the root of your repository.

### Q2: Is the filename case-sensitive?

Yes. It must be exactly `copilot-instructions.md` (lowercase).

### Q3: Do I need to enable anything in VS Code?

Check that **Code Generation: Use Instruction Files** is enabled in Settings. For prompt files, also enable `"chat.promptFiles": true`.

### Q4: Do instructions work with GitHub.com Copilot Chat?

Yes. Instructions are loaded for Copilot Chat in VS Code, Visual Studio, JetBrains IDEs, and on GitHub.com.

### Q5: Do instructions affect inline code completions (autocomplete)?

Minimally. Instructions have the strongest effect on **Copilot Chat**, **Coding Agent**, and **Code Review**. Inline completions primarily use the surrounding code context.

---

## File Types

### Q6: What's the difference between instructions and prompt files?

**Instructions** (`.instructions.md`) are passive rules auto-loaded based on file context. **Prompt files** (`.prompt.md`) are active task workflows triggered on-demand by the user.

### Q7: Can I have multiple copilot-instructions.md files?

No. Only one repo-wide file at `.github/copilot-instructions.md`. For additional scoping, use path-specific `.instructions.md` files in `.github/instructions/`.

### Q8: What's the format of the applyTo field?

A glob pattern in quotes:
```yaml
---
applyTo: "src/components/**/*.tsx"
---
```

### Q9: Can applyTo match multiple patterns?

Use brace expansion: `"**/*.{test.ts,test.tsx,spec.ts}"` or create multiple instruction files.

### Q10: Do instructions work with AGENTS.md?

Yes. The Copilot Coding Agent reads both `copilot-instructions.md` AND `AGENTS.md`. They complement each other.

---

## Writing & Content

### Q11: How long should my instruction file be?

- **copilot-instructions.md**: 500-1,000 words ideal
- **Path-specific files**: 200-500 words each
- **Code Review limit**: ~4,000 characters total

### Q12: What's the most important section to include?

**Commands** — build, test, lint commands. This is the single highest-impact section, especially for the Coding Agent.

### Q13: Should I include code examples in instructions?

Short inline examples (1-3 lines) are fine. Long examples waste tokens — put those in prompt files instead.

### Q14: Can I use Markdown formatting?

Yes. Headings, bullet lists, code blocks, tables — all supported. Bullet lists are parsed most reliably.

### Q15: Should I include a project structure section?

Yes, but keep it to 5-10 lines (top-level directories only). Don't list every file.

---

## Behavior & Effectiveness

### Q16: Will Copilot always follow my instructions?

No. Instructions are strong suggestions, not guarantees. Compliance rate is ~85-95% for clear, specific instructions. Vague instructions have much lower compliance.

### Q17: Why does Copilot sometimes ignore my instructions?

Common causes:
- Instruction is too vague ("write clean code")
- Instruction is buried in the middle of a long file
- Instruction conflicts with the existing code pattern
- Context window is full (too many open files + long conversation)

### Q18: Do instructions apply to code review?

Yes, but with a ~4,000 character limit. Review rules beyond this limit are silently ignored.

### Q19: How do I know if my instructions are working?

1. Open Copilot Chat → check References panel for your file
2. Ask Copilot about your project conventions
3. Ask Copilot to generate code → verify compliance with your rules

---

## Team & Organization

### Q20: Can I set instructions for my entire organization?

Yes. Go to Organization Settings → Copilot → Custom Instructions. These apply to all repos in the org.

### Q21: Which level takes priority?

Personal > Path-specific > Repository > Organization. All are combined; the most specific wins in conflicts.

### Q22: Should I put instructions in CODEOWNERS?

Yes. Assign tech lead or platform team as owners for `.github/copilot-instructions.md` and `.github/instructions/`.

### Q23: How do we handle instruction changes?

Treat them like code changes: PRs with review, testing, and communication to the team.

---

## Advanced

### Q24: Can I exclude specific agents from seeing instructions?

Yes. Use the `excludeAgent` field in path-specific instructions:
```yaml
---
applyTo: "**/*.ts"
excludeAgent: "code-review"
---
```
Valid values: `"code-review"`, `"coding-agent"`.

### Q25: How do I migrate from .cursorrules or CLAUDE.md?

1. Create `AGENTS.md` with your universal rules
2. Create `.github/copilot-instructions.md` with Copilot-specific additions
3. Keep old files as thin redirects: "See AGENTS.md for conventions"
4. See the [Migration Guide](migration-guide.md) for detailed steps

---

*Next: [Instruction Audit Checklist](audit-checklist.md) →*
