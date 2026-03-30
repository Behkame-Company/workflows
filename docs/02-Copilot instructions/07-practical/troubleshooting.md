# Troubleshooting Copilot Instructions

> When instructions don't work as expected — diagnosis and fixes.

---

## Problem: Copilot Ignores My Instructions

### Diagnosis Steps

1. **Check References Panel**
   - Open Copilot Chat
   - Send any message
   - Expand "References" at the top of the response
   - Is `copilot-instructions.md` listed?
   - If NO → File not loading (see below)

2. **Check File Location**
   - File MUST be at `.github/copilot-instructions.md`
   - Not in repo root, not in `docs/`, not in `src/`
   - Case-sensitive filename

3. **Check IDE Setting**
   - VS Code: Settings → search "instruction file"
   - Verify "Code Generation: Use Instruction Files" is checked
   - Verify "Chat: Code Generation Instructions" includes your file

4. **Check File Content**
   - Is the file empty?
   - Is all content in HTML comments `<!-- -->`?
   - Is the file valid Markdown?

### Common Causes

| Cause | Fix |
|-------|-----|
| Wrong file location | Move to `.github/copilot-instructions.md` |
| Setting disabled | Enable in VS Code settings |
| File too large | Trim to under 1,000 words |
| Instructions too vague | Make specific and actionable |
| Instructions buried in middle | Move critical rules to top or bottom |

---

## Problem: Path-Specific Instructions Not Loading

### Diagnosis Steps

1. **Check `applyTo` pattern**
   ```bash
   # Test if your glob matches the expected files
   find . -path "./src/components/**/*.tsx" -type f | head -5
   ```

2. **Check file location**
   - Must be in `.github/instructions/`
   - Must end with `.instructions.md`

3. **Check frontmatter**
   ```yaml
   ---
   applyTo: "src/components/**/*.tsx"   # Must be in quotes
   ---
   ```
   - Must be the very first thing in the file
   - Must have `---` delimiters
   - Pattern must be in quotes

4. **Check matching file is open**
   - Path-specific instructions only load when you're working on a matching file
   - Open a file that matches the pattern, then check References

### Common Causes

| Cause | Fix |
|-------|-----|
| Glob doesn't match | Test glob with file search |
| Missing frontmatter | Add `---` delimited YAML at top |
| Wrong directory | Move to `.github/instructions/` |
| Pattern in wrong format | Use string, not array |
| File not open | Open a matching file first |

---

## Problem: Copilot Follows Instructions Inconsistently

### Why This Happens

Copilot is a probabilistic model. Instructions are "strong suggestions," not hard constraints. Inconsistency increases with:

- Ambiguous instructions
- Conflicting instructions across files
- Long instruction files (middle content ignored)
- Context competition (many files open, long conversation)

### Diagnosis Steps

1. **Check for contradictions**
   ```bash
   # Find potentially conflicting terms
   grep -ri "default export" .github/
   grep -ri "named export" .github/
   ```

2. **Check instruction position**
   - Is the ignored rule in the middle of a long file?
   - Move it to the top or bottom

3. **Check instruction clarity**
   - Is it vague? ("try to use") → Make imperative ("use")
   - Is it conditional? ("when appropriate") → Remove conditions
   - Is it novel? (something Copilot naturally wouldn't do) → Add emphasis and examples

### Fixes

| Issue | Fix |
|-------|-----|
| Instruction in middle of long file | Move to top or into "Do Not" section |
| Hedging language | Change to imperative voice |
| Competing with codebase patterns | Add "even if existing code uses X" |
| Too many rules | Reduce to 5-10 highest-impact rules |

---

## Problem: Code Review Ignores Instructions

### Key Constraint

Code Review processes approximately **4,000 characters** of instructions. Content beyond that is silently ignored.

### Diagnosis Steps

1. Check file size: `wc -c .github/copilot-instructions.md`
2. If over 4,000 characters, the tail is being ignored
3. Check if review-critical rules are in the first 4,000 characters

### Fixes

- Move review-critical rules to the TOP of the file
- Use separate path-specific files for review-focused rules
- Use `excludeAgent: "code-review"` on instructions that are only for Chat/Agent

---

## Problem: Coding Agent Fails

### Common Failure Modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| Wrong package manager | Agent guessed npm, you use pnpm | Specify in Commands section |
| Tests not run | No verification steps in instructions | Add Validation section |
| Wrong test framework | Agent used Jest, you use Vitest | Specify test framework |
| Creates wrong file structure | Structure section missing or wrong | Update Structure section |
| Modifies protected files | No scope boundaries | Add "Do Not modify" list |
| PR is broken | No build verification | Add "run build" to Validation |

### Agent-Specific Debugging

1. Check the agent's PR for what it tried to do
2. Read the Actions logs for command output
3. Verify instruction files are committed (agent reads from HEAD)
4. Test the commands from instructions locally

---

## Problem: Prompt Files Not Available

### Diagnosis Steps

1. **Check VS Code setting**
   - Settings → search "prompt files"
   - Enable `"chat.promptFiles": true`

2. **Check file location**
   - Must be in `.github/prompts/`
   - Must end with `.prompt.md`

3. **Check frontmatter (if present)**
   - YAML must be valid
   - `---` delimiters must be present

---

## Problem: Instructions Conflict Between Tools

### When Using Copilot + Claude Code + Cursor

Each tool reads different files:
- Copilot: `.github/copilot-instructions.md` + `.instructions.md`
- Claude: `CLAUDE.md`
- Cursor: `.cursorrules`

### Fix: Use AGENTS.md as Source of Truth

```markdown
# AGENTS.md (read by all tools)
[Universal conventions and commands]

# .github/copilot-instructions.md
See AGENTS.md for project conventions.
[Copilot-specific additions only]

# CLAUDE.md
See AGENTS.md for project conventions.
[Claude-specific additions only]
```

---

## Debug Checklist

When instructions aren't working:

- [ ] File exists in correct location
- [ ] File is committed to the repository (Coding Agent reads from git)
- [ ] IDE setting enabled for instruction files
- [ ] File is under size limit (1,000 words / 4,000 chars for review)
- [ ] Instructions are specific and actionable
- [ ] No contradictions between instruction files
- [ ] Instructions are at the top or bottom of file (not buried in middle)
- [ ] Tested by asking Copilot a question the instruction addresses
- [ ] References panel shows the instruction file is loaded
