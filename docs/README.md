# Meta-Prompting Framework & AI-Assisted Development Guide

> A team-lead reference for building structured, repeatable AI-assisted development workflows — primarily with **GitHub Copilot CLI** and the Copilot Coding Agent.

---

## Why This Guide Exists

AI coding tools are only as good as the context they receive. Without structured instructions, every session starts from zero — your AI assistant doesn't know your conventions, your architecture, or how to validate its own work.

This documentation provides a **complete framework** for:

- **Instructing AI tools** with persistent, repo-level configuration files
- **Maintaining context** across sessions using memory bank patterns
- **Meta-prompting** — teaching AI *how to think* about your codebase, not just *what to do*
- **Cross-tool compatibility** so your team can use multiple AI assistants with shared standards

---

## Table of Contents

### 1. Foundations
- [What Is Meta-Prompting?](01-foundations/what-is-meta-prompting.md) — Core concepts and why it matters
- [Context Engineering](01-foundations/context-engineering.md) — From prompt engineering to context engineering
- [Instruction Files Overview](01-foundations/instruction-files-overview.md) — The landscape of AI instruction files

### 2. GitHub Copilot Instructions
- [copilot-instructions.md Guide](02-copilot-instructions/copilot-instructions-guide.md) — Repository-wide custom instructions
- [Path-Specific Instructions](02-copilot-instructions/path-specific-instructions.md) — Scoped rules with `.instructions.md`
- [Prompt Files](02-copilot-instructions/prompt-files.md) — Reusable `.prompt.md` files

### 3. AGENTS.md (Cross-Tool Standard)
- [AGENTS.md Guide](03-agents-md/agents-md-guide.md) — The universal AI agent instruction file

### 4. Memory Bank for Teams
- [Memory Bank for Teams](04-memory-bank/memory-bank-for-teams.md) — Persistent project context across sessions

### 5. Meta-Prompting Framework
- [Framework Architecture](05-meta-prompting-framework/framework-architecture.md) — Conductor-expert patterns and modular design
- [Building Your Framework](05-meta-prompting-framework/building-your-framework.md) — Step-by-step from scratch
- [Advanced Patterns](05-meta-prompting-framework/advanced-patterns.md) — Recursive refinement and multi-agent orchestration

### 6. Spec-Driven Development
- [Spec-Driven Overview](06-spec-driven-development/spec-driven-overview.md) — Brief introduction to specification-first workflow

### 7. Templates
- [copilot-instructions.md Template](07-templates/copilot-instructions-template.md)
- [AGENTS.md Template](07-templates/agents-md-template.md)
- [Path-Specific Instructions Template](07-templates/path-instructions-template.md)
- [Memory Bank Template](07-templates/memory-bank-template.md)
- [Prompt File Template](07-templates/prompt-file-template.md)

### 8. Reference
- [Tool Comparison](08-reference/tool-comparison.md) — copilot-instructions.md vs CLAUDE.md vs .cursorrules vs AGENTS.md
- [Resources & Further Reading](08-reference/resources.md)

---

## Deep-Dive Educational Sections

| Section | Documents | Focus |
|---------|-----------|-------|
| [Copilot Instructions](../Copilot%20instructions/) | 34 docs | Complete guide to copilot-instructions.md, path-specific, and prompt files |
| [AGENTS.md](../03-AGENTS.md/) | 31 docs | Cross-tool AGENTS.md standard — deep dives, compatibility, enterprise |
| [Spec-Driven Development](../04-Spec-Driven%20Development/) | 44 docs | Full SDD methodology — workflow, writing specs, tools, anti-patterns |

---

## Quick Start

1. **New to this?** Start with [What Is Meta-Prompting?](01-foundations/what-is-meta-prompting.md)
2. **Setting up Copilot?** Jump to [copilot-instructions.md Guide](02-copilot-instructions/copilot-instructions-guide.md)
3. **Multi-tool team?** Read [AGENTS.md Guide](03-agents-md/agents-md-guide.md) first
4. **Want templates now?** Go to [Templates](07-templates/)
5. **Building a full framework?** Follow [Building Your Framework](05-meta-prompting-framework/building-your-framework.md)
6. **Learning SDD?** Start with [What Is SDD?](../04-Spec-Driven%20Development/01-fundamentals/what-is-sdd.md)

---

## Conventions Used

| Icon | Meaning |
|------|---------|
| 💡 | Tip or recommendation |
| ⚠️ | Warning or common pitfall |
| 📋 | Template or example available |
| 🔗 | Cross-reference to another section |

---

*Last updated: March 2026*
