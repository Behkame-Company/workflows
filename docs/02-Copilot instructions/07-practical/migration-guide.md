# Migration Guide

> Migrating from .cursorrules, CLAUDE.md, or other AI instruction formats to Copilot instructions.

---

## Migration Overview

| Source Format | Target | Difficulty |
|---------------|--------|------------|
| `.cursorrules` → Copilot | Low | Same content, different location |
| `CLAUDE.md` → Copilot | Low | Same content, split into layers |
| `.cursorrules` + `CLAUDE.md` → Unified | Medium | Merge + deduplicate |
| No instructions → Copilot | Medium | Write from scratch |
| Custom format → Copilot | Varies | Map content to Copilot structure |

---

## Step 1: Audit Current State

Before migrating, inventory what you have:

```bash
# Find all existing AI instruction files
find . -name ".cursorrules" -o -name "CLAUDE.md" -o -name "AGENTS.md" \
       -o -name "*.instructions.md" -o -name "copilot-instructions.md" \
       -o -name ".cursorignore" -o -name "GEMINI.md" 2>/dev/null
```

```powershell
# PowerShell equivalent
Get-ChildItem -Recurse -Include ".cursorrules","CLAUDE.md","AGENTS.md","*.instructions.md","copilot-instructions.md" -ErrorAction SilentlyContinue
```

---

## Step 2: Map Content

### From .cursorrules

`.cursorrules` is a flat Markdown file in the repo root. Map sections to Copilot files:

| .cursorrules Content | Copilot Destination |
|---------------------|---------------------|
| Project description | copilot-instructions.md → ## Project |
| Universal conventions | copilot-instructions.md → ## Conventions |
| Frontend rules | frontend.instructions.md |
| Backend rules | backend.instructions.md |
| Commands/scripts | copilot-instructions.md → ## Commands |
| "Do not" rules | copilot-instructions.md → ## Do Not |

### From CLAUDE.md

`CLAUDE.md` supports directory hierarchy. Map to Copilot's path-specific system:

| CLAUDE.md | Copilot Equivalent |
|-----------|---------------------|
| Root `CLAUDE.md` | `.github/copilot-instructions.md` |
| `src/frontend/CLAUDE.md` | `.github/instructions/frontend.instructions.md` (applyTo: `src/frontend/**`) |
| `src/backend/CLAUDE.md` | `.github/instructions/backend.instructions.md` (applyTo: `src/backend/**`) |

### Creating AGENTS.md (Universal)

If you want cross-tool compatibility, create AGENTS.md alongside Copilot files:

```markdown
# AGENTS.md (read by Copilot + Claude + Cursor + Codex + Gemini)
[Universal content that applies to all tools]
```

---

## Step 3: Create Copilot Files

### 3a: Create copilot-instructions.md

```bash
mkdir -p .github
```

Extract universal content from your source file into `.github/copilot-instructions.md`:

```markdown
# Copilot Instructions

## Project
[From your existing description]

## Commands
[Extract all commands — this section is NEW and critical for Copilot]

## Conventions
[From your existing conventions]

## Do Not
[From your existing restrictions]

## Validation
[NEW — add build/test verification steps]
```

💡 **The Commands and Validation sections may not exist in your .cursorrules or CLAUDE.md.** These are critical for Copilot (especially the Coding Agent) — add them during migration.

### 3b: Create Path-Specific Files

For each domain-specific section in your source:

```bash
mkdir -p .github/instructions
```

```markdown
---
applyTo: "src/components/**/*.tsx"
---
# Frontend Instructions
[Extracted frontend-specific rules]
```

### 3c: Migrate Prompt Patterns

If your source file contains "how to do X" sections (task workflows), migrate those to prompt files:

```bash
mkdir -p .github/prompts
```

```markdown
---
name: add-feature
description: Add a new feature following project patterns
---
[Workflow steps extracted from your source]
```

---

## Step 4: Handle Cross-Tool Compatibility

### Option A: AGENTS.md as Primary (Recommended for Multi-Tool Teams)

```
AGENTS.md                              # Universal rules (all tools read)
.github/copilot-instructions.md        # "See AGENTS.md" + Copilot extras
CLAUDE.md                              # "See AGENTS.md" + Claude extras
```

```markdown
# .github/copilot-instructions.md
See AGENTS.md for project conventions and commands.

## Copilot-Specific
- Code review focus: type safety and error handling
- Coding agent: always run full test suite before creating PR
```

```markdown
# CLAUDE.md
See AGENTS.md for project conventions and commands.

## Claude-Specific
- Use TodoWrite for multi-step tasks
- Prefer edit over rewrite for small changes
```

### Option B: Copilot-Only (Single Tool)

Remove old files and use only Copilot's instruction system:

```
.github/copilot-instructions.md          # All universal rules
.github/instructions/*.instructions.md   # Domain-specific rules
.github/prompts/*.prompt.md              # Task workflows
```

---

## Step 5: Verify Migration

### Test Checklist

- [ ] Open VS Code, open the project
- [ ] Open Copilot Chat → check References → verify files are loaded
- [ ] Ask "What are the project conventions?" → verify answer matches
- [ ] Generate code → verify it follows conventions
- [ ] If multi-tool: test with Claude Code / Cursor to verify AGENTS.md works
- [ ] Run `pnpm test` (or equivalent) to confirm no functional impact

### Cleanup

After verification:

```bash
# Option 1: Remove old files (single-tool)
git rm .cursorrules
git rm CLAUDE.md
git commit -m "chore: migrate AI instructions to Copilot format"

# Option 2: Convert to redirects (multi-tool)
echo "See AGENTS.md for project conventions." > .cursorrules
echo "See AGENTS.md for project conventions." > CLAUDE.md
git add .cursorrules CLAUDE.md
git commit -m "chore: redirect tool-specific files to AGENTS.md"
```

---

## Common Migration Pitfalls

| Pitfall | Fix |
|---------|-----|
| Copying entire .cursorrules verbatim | Split into repo-wide + path-specific |
| Missing Commands section | Always add commands — critical for Coding Agent |
| Missing Validation section | Add build/test verification steps |
| Duplicated content across AGENTS.md + copilot-instructions.md | Single source of truth in AGENTS.md |
| Not testing after migration | Always verify Copilot reads and follows the new files |

---

## Migration Checklist

- [ ] Inventoried existing instruction files
- [ ] Mapped content to Copilot file structure
- [ ] Created `.github/copilot-instructions.md` with Commands and Validation
- [ ] Created path-specific `.instructions.md` files for domains
- [ ] Created `.prompt.md` files for task workflows
- [ ] Created/updated `AGENTS.md` for cross-tool compatibility
- [ ] Verified files load in Copilot References
- [ ] Tested code generation follows new instructions
- [ ] Cleaned up or redirected old instruction files
- [ ] Committed with clear message
- [ ] Communicated change to team

---

*← Back to [README](../README.md)*
