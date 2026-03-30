# AGENTS.md vs CLAUDE.md vs .cursorrules

> A detailed side-by-side comparison of all major AI agent instruction formats.

---

## Quick Comparison

| Feature | AGENTS.md | CLAUDE.md | .cursorrules | copilot-instructions.md |
|---------|-----------|-----------|-------------|------------------------|
| **Format** | Markdown | Markdown | Plain text | Markdown |
| **Standard body** | AAIF / Linux Foundation | Anthropic (proprietary) | Cursor (proprietary) | GitHub (proprietary) |
| **Location** | Root + subdirectories | Root + subdirectories + ~/.claude/ | Root only (legacy) | .github/ |
| **Hierarchy** | ✅ Nearest wins | ✅ Merges all levels | ❌ Single file only | ✅ Path-specific rules |
| **Cross-tool** | ✅ Universal | ❌ Claude only | ❌ Cursor only | ❌ Copilot only |
| **Max size** | ~300 lines recommended | No hard limit (but ~1,500 recommended) | No hard limit | ~500-1,000 words |
| **Auto-loaded** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Tools supported** | 10+ | 1 (Claude) | 1 (Cursor) | 1 (Copilot) |

---

## Format Deep Dive

For detailed per-tool support information, feature comparisons, and behavioral differences, see [Cross-Tool Matrix](cross-tool-matrix.md).

---

## When to Use Which

### Decision Matrix

| Scenario | Use |
|----------|-----|
| Open-source project (any contributors) | **AGENTS.md** only |
| Team uses only Copilot | **copilot-instructions.md** + **AGENTS.md** |
| Team uses only Claude | **CLAUDE.md** + **AGENTS.md** |
| Team uses only Cursor | **.cursor/rules/** + **AGENTS.md** |
| Multi-tool team | **AGENTS.md** (primary) + tool-specific (supplements) |
| Monorepo, multi-tool | **AGENTS.md** hierarchy (primary) + tool-specific per needs |

### The Universal Rule

> **Always have an AGENTS.md.** Add tool-specific files only for tool-specific features.

---

## Feature Availability by File

| Feature | AGENTS.md | CLAUDE.md | .cursor/rules/ | copilot-instructions.md |
|---------|:---------:|:---------:|:---------------:|:----------------------:|
| Build/test commands | ✅ | ✅ | ✅ | ✅ |
| Coding conventions | ✅ | ✅ | ✅ | ✅ |
| Boundaries (NEVER rules) | ✅ | ✅ | ✅ | ✅ |
| Per-directory hierarchy | ✅ | ✅ | ❌ | ❌ |
| Glob-based scoping | ❌ | ❌ | ✅ | ✅ |
| User-level preferences | ❌ | ✅ (~/.claude/) | ❌ | ✅ (personal settings) |
| Org-level standards | ❌ | ❌ | ❌ | ✅ (Enterprise) |
| Skills / sub-agents | ❌ | ✅ | ❌ | ❌ |
| Memory persistence | ❌ | ✅ | ❌ | ❌ |
| Cross-tool compatible | ✅ | ❌ | ❌ | ❌ |

---

## Migration Priority

If you're starting fresh:
1. Create AGENTS.md first (universal foundation)
2. Add tool-specific files only when you need features unique to that tool

If you have existing tool-specific files:
1. Keep them for tool-specific features
2. Add AGENTS.md with universal rules
3. Avoid duplicating rules between files
4. See [Migration Guide](migration-guide.md) for step-by-step instructions
