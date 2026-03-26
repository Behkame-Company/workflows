# AGENTS.md Frequently Asked Questions

> 20 answers to the most common questions about AGENTS.md.

---

## General

### 1. What is AGENTS.md?

AGENTS.md is a Markdown file placed at the root of a code repository that provides instructions for AI coding agents. It's the AI equivalent of a README — while README.md onboards human developers, AGENTS.md onboards AI agents.

It's an open standard governed by the Agentic AI Foundation (AAIF) under the Linux Foundation, supported by 10+ tools including GitHub Copilot, OpenAI Codex, Claude Code, Cursor, and Gemini CLI.

---

### 2. Is AGENTS.md just for GitHub Copilot?

No. AGENTS.md is a **universal, cross-tool standard**. It works with GitHub Copilot, OpenAI Codex, Claude Code, Cursor, Gemini CLI/Jules, Goose, Continue.dev, Zed, and more. That's the key advantage — write once, use with any AI tool.

---

### 3. Is AGENTS.md case-sensitive?

Yes. The file must be named `AGENTS.md` (all uppercase). `agents.md`, `Agents.md`, or `AGENTS.MD` may not be recognized by all tools.

---

### 4. Where should I put AGENTS.md?

At the **root** of your repository. For monorepos, you can also place additional AGENTS.md files in subdirectories for package-specific instructions.

---

### 5. How is AGENTS.md different from README.md?

| Aspect | README.md | AGENTS.md |
|--------|-----------|-----------|
| Audience | Human developers | AI coding agents |
| Purpose | Understand the project | Follow coding rules |
| Content | History, screenshots, examples | Commands, rules, boundaries |
| Length | As long as needed | Under 200 lines |

---

### 6. Do I still need README.md if I have AGENTS.md?

Yes, absolutely. README.md is for humans; AGENTS.md is for agents. They serve different purposes and contain different content. Don't duplicate between them.

---

## Setup & Configuration

### 7. How long should AGENTS.md be?

**50-100 lines** is the sweet spot. Research shows:
- Under 50 lines: ~90% agent compliance
- 50-100 lines: ~82% compliance
- 100-200 lines: ~70% compliance
- 300+ lines: ~40% compliance

Start small and add rules as needed.

---

### 8. What sections should I include?

At minimum:
1. **Setup** — Commands to build, test, and run
2. **Conventions** — Your top 5-10 coding rules
3. **Boundaries** — What agents must NEVER do

Recommended additions: Stack, Structure, Testing, Git Workflow.

---

### 9. Do I need special syntax or formatting?

No. Standard Markdown. No YAML frontmatter required, no schemas, no custom syntax. Any developer can write it.

---

### 10. How do I test if my AGENTS.md is working?

1. Ask the agent: "What are the project conventions?"
2. Assign a task and check if it follows your rules
3. Test a boundary: does the agent refuse to modify protected files?
4. Check commands: does the agent run the exact commands you specified?

---

## Multi-Tool & Compatibility

### 11. I use Copilot and have copilot-instructions.md. Do I also need AGENTS.md?

**Recommended: yes.** Use copilot-instructions.md for Copilot-specific features (path-specific rules, code review instructions) and AGENTS.md for universal rules that any tool should follow. This gives you portability and consistency.

---

### 12. I use Claude Code with CLAUDE.md. Should I add AGENTS.md?

**Recommended: yes.** Claude reads AGENTS.md as a fallback. Having both means:
- CLAUDE.md: Claude-specific features (skills, memory)
- AGENTS.md: Universal rules for any other tool or contributor

---

### 13. Can I use AGENTS.md alongside .cursorrules?

Yes. Cursor reads AGENTS.md alongside .cursorrules. Put universal rules in AGENTS.md and Cursor-specific scoped rules in .cursor/rules/*.mdc.

---

### 14. What if two tools interpret AGENTS.md differently?

Keep AGENTS.md simple and focused on universally applicable instructions. Avoid edge cases. Test with each tool your team uses. If a tool needs specific behavior, use that tool's native format for those specific rules.

---

## Maintenance & Governance

### 15. How often should I update AGENTS.md?

Update when:
- You change frameworks, tools, or build commands
- An agent makes a mistake that a rule could prevent
- You restructure the project
- Dependencies have major version updates

**Minimum**: quarterly review. **Ideal**: update whenever conventions change.

---

### 16. Who should own AGENTS.md?

| Layer | Owner |
|-------|-------|
| Security boundaries | Security team |
| Conventions and patterns | Tech lead / senior devs |
| Setup commands | Any developer (PR reviewed) |

Use CODEOWNERS to enforce review requirements.

---

### 17. Should AGENTS.md be in .gitignore?

**Never.** AGENTS.md must be committed to the repository. AI tools read it from the repo. It's part of your project's configuration, just like README.md or package.json.

---

## Advanced

### 18. Can I use AGENTS.md for non-coding tasks?

Yes. You can include sections for documentation, design, or DevOps agents. However, keep the file focused on what agents do most: writing and modifying code.

---

### 19. Does AGENTS.md work with GitHub Copilot Coding Agent (issue-assigned)?

Yes. When you assign an issue to Copilot, the Coding Agent reads AGENTS.md (along with copilot-instructions.md) to understand project conventions before generating a PR.

---

### 20. Can agents modify AGENTS.md itself?

They can, but they shouldn't. Add a boundary:

```markdown
## Boundaries
- NEVER modify AGENTS.md, README.md, or .github/ files
```

AGENTS.md should only be modified by humans through reviewed PRs.

---

## Quick Decision Guide

| Question | Answer |
|----------|--------|
| Should I create AGENTS.md? | Yes — any project using AI tools benefits |
| Should I delete other instruction files? | No — keep tool-specific files for tool-specific features |
| How long to write? | 15-30 minutes for a good first version |
| Where to start? | Setup commands + 5 boundaries |
| When to split? | When root file exceeds 150 lines (monorepo) |
| What if it's not working? | See [Troubleshooting Guide](troubleshooting.md) |

---

*Next: [Audit Checklist](audit-checklist.md) →*
