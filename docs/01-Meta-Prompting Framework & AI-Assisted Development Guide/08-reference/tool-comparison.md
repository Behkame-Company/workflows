# Tool Comparison

> copilot-instructions.md vs CLAUDE.md vs .cursorrules vs AGENTS.md — side by side.

---

## Quick Comparison

| Feature | copilot-instructions.md | CLAUDE.md | .cursorrules / .mdc | AGENTS.md |
|---------|------------------------|-----------|---------------------|-----------|
| **Primary Tool** | GitHub Copilot | Claude Code | Cursor | All tools |
| **Location** | `.github/` | Project root, `~/.claude/`, subdirs | Project root, `.cursor/rules/` | Repo root, any subdir |
| **Auto-loaded** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes (by agent) |
| **Path scoping** | ✅ `applyTo` glob | ✅ Directory hierarchy | ✅ `.mdc` glob | ✅ Directory hierarchy |
| **Hierarchical** | Org → Repo → Path | Global → Project → Dir | Project only | Root → Subdir |
| **Multi-tool** | ❌ Copilot only | ❌ Claude only | ❌ Cursor only | ✅ Universal |
| **Prompt files** | ✅ `.prompt.md` | ❌ | ❌ | ❌ |
| **Code Review** | ✅ Copilot Review | ❌ | ❌ | ❌ |
| **Format** | Markdown + frontmatter | Markdown | Plain text / MDC | Markdown |
| **Governance** | GitHub | Anthropic | Cursor/Anysphere | Linux Foundation (AAIF) |

---

## Detailed Breakdown

### GitHub Copilot: `copilot-instructions.md`

**Strengths:**
- Best integration with GitHub ecosystem (Chat, Coding Agent, Code Review)
- Path-specific scoping with `.instructions.md` + `applyTo` globs
- Reusable prompt files (`.prompt.md`)
- Organization-level instruction inheritance
- Auto-generation by Copilot Coding Agent

**Limitations:**
- Only used by GitHub Copilot products
- No built-in memory/persistence beyond instruction files
- Cannot reference external tools or MCP integrations

**Best for:** Teams standardized on GitHub Copilot across the organization.

---

### Claude Code: `CLAUDE.md`

**Strengths:**
- Best hierarchical config system (global `~/.claude/CLAUDE.md` → project → directory)
- Persistent memory via claude-mem plugin or manual memory bank
- Skills and subagent support for complex workflows
- Token-aware: Claude can report how much context is consumed

**Limitations:**
- Only used by Claude Code
- No built-in path-scoping (uses directory hierarchy instead)
- No equivalent to prompt files (use slash commands instead)

**Best for:** Teams primarily using Claude Code, especially for complex multi-session projects.

---

### Cursor: `.cursorrules` / `.cursor/rules/*.mdc`

**Strengths:**
- Simplest setup: drop a text file in root and it works
- Modern `.mdc` files support powerful conditional scoping
- Falls back to AGENTS.md if `.cursorrules` is missing
- Seamless with Cursor's inline AI features

**Limitations:**
- No organization-level inheritance
- No global config across projects (project-only)
- Legacy `.cursorrules` has no scoping; must use `.mdc` for that

**Best for:** Individual developers or small teams using Cursor as their primary editor.

---

### Universal: `AGENTS.md`

**Strengths:**
- Supported by all major AI coding tools
- Vendor-neutral, open standard (Linux Foundation governance)
- Used in 60,000+ repos and growing
- Works as a fallback for tools without their own config file
- Monorepo-friendly with directory hierarchy

**Limitations:**
- No path-specific scoping (only directory-level)
- No advanced features like prompt files or code review integration
- Newer standard — some tools may have incomplete support

**Best for:** Open source projects, multi-tool teams, and future-proofing.

---

## Strategy Matrix

### Single-Tool Teams

| If your team uses... | Primary file | Also create |
|----------------------|--------------|-------------|
| GitHub Copilot | `copilot-instructions.md` + `.instructions.md` | `AGENTS.md` (compatibility) |
| Claude Code | `CLAUDE.md` | `AGENTS.md` (compatibility) |
| Cursor | `.cursor/rules/*.mdc` | `AGENTS.md` (compatibility) |

### Multi-Tool Teams

```
AGENTS.md                          ← Universal rules (all tools read)
    +
copilot-instructions.md            ← Copilot-specific features
    +
CLAUDE.md                          ← Claude-specific features (optional)
```

**Rule of thumb**: Put universal conventions in `AGENTS.md`. Put tool-specific features (path scoping, prompt files, code review) in the tool's native file.

### Open Source Projects

```
AGENTS.md                          ← Primary (everyone can use any tool)
    +
copilot-instructions.md            ← If maintainers use Copilot
```

---

## Recommendation for Teams Starting Fresh

1. **Start with `copilot-instructions.md`** — highest impact for Copilot users
2. **Add `AGENTS.md`** — same core rules, cross-tool compatible
3. **Add `.instructions.md` files** — when domain boundaries become clear
4. **Add `.prompt.md` files** — for workflows the team repeats often
5. **Add `memory-bank/`** — when session continuity becomes important

This layered adoption lets you build incrementally without overwhelming the team.
