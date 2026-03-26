# Cross-Tool Compatibility Matrix

> Which AI coding tools read AGENTS.md, how they process it, and what to expect.

---

## The Compatibility Matrix (March 2026)

| Tool | Reads AGENTS.md | How It's Used | Location |
|------|:---------------:|---------------|----------|
| **GitHub Copilot Coding Agent** | ✅ | Auto-loaded for tasks; used alongside copilot-instructions.md | Root |
| **GitHub Copilot Chat** | ✅ | Loaded as context in conversations | Root |
| **OpenAI Codex CLI** | ✅ | Primary instruction file; auto-loaded | Root + subdirectories |
| **Claude Code** | ✅ (fallback) | Read if CLAUDE.md not found; CLAUDE.md takes priority | Root |
| **Cursor** | ✅ | Read alongside .cursorrules / .cursor/rules/ | Root |
| **Google Gemini CLI** | ✅ | Auto-loaded for project context | Root |
| **Google Jules** | ✅ | Read for task execution context | Root |
| **Goose (Block)** | ✅ | Primary instruction file | Root |
| **Continue.dev** | ✅ | Project context for suggestions | Root |
| **Zed AI** | ✅ | Project context | Root |
| **Windsurf** | ⚠️ Partial | May read; uses own `.windsurfrules` primarily | Root |
| **Amazon Q Developer** | ⚠️ Partial | Reads as context; has own `.amazonq/` config | Root |

### Legend
- ✅ Fully supported — reads and follows AGENTS.md
- ⚠️ Partial — may read but prefers own format
- ❌ Not supported

---

## Per-Tool Details

### GitHub Copilot (Coding Agent + Chat)

```
Priority:
1. User prompt / issue description
2. .github/copilot-instructions.md (repo-wide)
3. **/*.instructions.md (path-specific)
4. AGENTS.md (root)
5. Codebase patterns
```

**Key behaviors:**
- Copilot reads AGENTS.md alongside its own instruction files
- Copilot Coding Agent uses AGENTS.md for assigned issues → PR generation
- copilot-instructions.md and AGENTS.md are **complementary**, not conflicting
- Code review has a separate ~4,000 character limit for instructions

**Recommendation**: Use both `.github/copilot-instructions.md` (for Copilot-specific features like path-specific rules) and `AGENTS.md` (for universal rules). Avoid duplicating content.

### OpenAI Codex CLI

```
Priority:
1. User command / prompt
2. Nearest AGENTS.md (directory hierarchy)
3. Root AGENTS.md (if not the same)
4. Codebase patterns
```

**Key behaviors:**
- AGENTS.md is the primary/only instruction file
- Full directory hierarchy support (nearest file wins)
- Most faithful implementation of the AGENTS.md standard
- Created the original specification

**Recommendation**: If your team uses Codex CLI, AGENTS.md is your only instruction file. Invest in it.

### Claude Code

```
Priority:
1. User prompt
2. ~/.claude/CLAUDE.md (user-level)
3. Project CLAUDE.md (root)
4. Subdirectory CLAUDE.md (nearest)
5. AGENTS.md (fallback if no CLAUDE.md)
6. Codebase patterns
```

**Key behaviors:**
- CLAUDE.md takes priority when present
- AGENTS.md is a fallback for projects without CLAUDE.md
- CLAUDE.md supports Claude-specific features (skills, sub-agents, memory)
- Hierarchical merging: all CLAUDE.md files in the tree are combined

**Recommendation**: Use AGENTS.md as the universal file. Add CLAUDE.md only if you need Claude-specific features.

### Cursor

```
Priority:
1. User prompt / chat context
2. .cursorrules (legacy, root-level)
3. .cursor/rules/*.mdc (scoped rules with YAML frontmatter)
4. AGENTS.md (root)
5. Codebase patterns
```

**Key behaviors:**
- Reads AGENTS.md from root
- .cursorrules and .cursor/rules/ take priority
- .cursor/rules/*.mdc supports scoped rules with `globs:` frontmatter
- Some teams report inconsistent AGENTS.md reading in Cursor

**Recommendation**: Use AGENTS.md for universal rules. Use .cursor/rules/ for Cursor-specific scoped rules.

### Gemini CLI / Jules

```
Priority:
1. User prompt / task description
2. AGENTS.md (root)
3. Codebase patterns
```

**Key behaviors:**
- Straightforward AGENTS.md support
- No tool-specific alternative file format
- Good compliance with commands and boundaries

---

## The Compatibility Strategy

### For Single-Tool Teams

If everyone uses the same tool, optimize for that tool:

| Your Tool | Primary File | Also Add AGENTS.md? |
|-----------|-------------|---------------------|
| Copilot | .github/copilot-instructions.md | ✅ Yes — for future portability |
| Codex | AGENTS.md | N/A — it is your primary file |
| Claude | CLAUDE.md | ✅ Yes — for contributors using other tools |
| Cursor | .cursor/rules/ | ✅ Yes — for contributors using other tools |

### For Multi-Tool Teams

If team members use different tools, adopt a layered strategy:

```
Layer 1: AGENTS.md (universal rules — everyone reads this)
Layer 2: Tool-specific files (optional, for tool-specific features)
```

```
AGENTS.md                        ← All tools read this
├── .github/copilot-instructions.md  ← Copilot users get path-specific rules
├── CLAUDE.md                        ← Claude users get memory/skills
└── .cursor/rules/                   ← Cursor users get IDE-specific rules
```

**Rule: Never put essential rules only in a tool-specific file.** If it's important, it goes in AGENTS.md.

---

## Testing Cross-Tool Compatibility

### Quick Test Matrix

| Test | Expected Result |
|------|----------------|
| Ask agent: "What are the project conventions?" | Should quote AGENTS.md content |
| Ask agent to create a function | Should follow AGENTS.md conventions |
| Ask agent to run tests | Should use exact command from AGENTS.md |
| Ask agent to modify protected directory | Should refuse per AGENTS.md boundary |

Run these tests with each tool your team uses to verify consistent behavior.

---

*Next: [AGENTS.md vs CLAUDE.md vs .cursorrules](comparison-guide.md) →*
