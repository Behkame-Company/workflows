# Instruction Audit Checklist

> Quarterly review template for keeping instruction files accurate and effective.

---

## How to Use This Checklist

Run this audit every **quarter** (set a recurring calendar event). Assign one person per repository to complete it. Takes about 30-60 minutes per repo.

---

## Section 1: File Inventory

List all instruction files in the repository:

| File | Exists? | Last Modified | Word Count |
|------|---------|---------------|------------|
| `.github/copilot-instructions.md` | | | |
| `.github/instructions/*.instructions.md` | | | |
| `.github/prompts/*.prompt.md` | | | |
| `AGENTS.md` | | | |

**Action items:**
- [ ] All files exist and are not empty
- [ ] Total word count across simultaneously-loaded files is under 2,000

---

## Section 2: Accuracy Check

### Commands

- [ ] `install` command works: `_______________`
- [ ] `dev` command starts the server: `_______________`
- [ ] `test` command runs tests: `_______________`
- [ ] `lint` command runs linter: `_______________`
- [ ] `build` command produces build: `_______________`
- [ ] All other documented commands work as described

### Project Description

- [ ] Stack description matches current reality
- [ ] Framework versions are correct
- [ ] Project status is accurate (production, beta, etc.)

### Structure

- [ ] Directory structure matches what's in the instructions
- [ ] No referenced directories have been renamed or deleted
- [ ] No new major directories are missing from the description

### Conventions

For each convention listed, verify:
- [ ] Convention is still followed by the team
- [ ] Convention hasn't been replaced by a different approach
- [ ] Convention is specific enough to be actionable

### Do Not Rules

- [ ] All "never do X" rules are still valid
- [ ] No new "never do" rules needed based on recent incidents
- [ ] No rules that are now handled by linter/CI instead

---

## Section 3: Effectiveness Test

For each major rule, test Copilot compliance:

| Instruction | Test Prompt | Copilot Complied? | Action |
|-------------|-------------|-------------------|--------|
| "Use TypeScript strict mode" | "Create a utility function" | ✅/❌ | |
| "Validate with Zod" | "Create an API endpoint" | ✅/❌ | |
| "Use project logger" | "Add error handling" | ✅/❌ | |
| [Your rule] | [Your test] | ✅/❌ | |

**For non-compliant rules:**
- Is the instruction too vague? → Rewrite
- Is it buried in the middle? → Move to top/bottom
- Is it conflicting with existing code? → Add explicit override note

---

## Section 4: Budget Check

- [ ] copilot-instructions.md: _____ words (target: <1,000)
- [ ] Each .instructions.md: _____ words (target: <500 each)
- [ ] Total simultaneous load: _____ words (target: <2,000)

**If over budget:**
- [ ] Remove rules that duplicate linter/CI checks
- [ ] Remove vague rules that scored low on effectiveness
- [ ] Move domain-specific rules to path-specific files
- [ ] Move examples to prompt files

---

## Section 5: Consistency Check

- [ ] No contradictions between repo-wide and path-specific files
- [ ] No duplicated rules across files
- [ ] Path-specific overrides are explicitly noted
- [ ] All `applyTo` patterns match current directory structure
- [ ] Prompt file references point to files that still exist

---

## Section 6: Security Check

- [ ] No secrets, credentials, or API keys in any instruction file
- [ ] No internal network details or infrastructure specifics
- [ ] No hidden Unicode characters (run scan)
- [ ] CODEOWNERS configured for `.github/` directory
- [ ] Branch protection enabled for instruction file changes

**Run security scan:**
```bash
# Check for hidden Unicode
grep -rP '[\x{200B}-\x{200F}\x{FEFF}\x{00AD}]' .github/ AGENTS.md

# Check for potential secrets
grep -riE '(password|secret|api.key|token|credential)\s*[:=]' .github/ AGENTS.md
```

---

## Section 7: Team Feedback

Gather from the team:
- [ ] Are developers aware instruction files exist?
- [ ] Do developers find Copilot's output useful?
- [ ] What does Copilot get wrong most often?
- [ ] Are there patterns Copilot should follow but doesn't?
- [ ] Are there instructions that are unnecessary or annoying?

**Feedback action items:**
1. ____________________________________________
2. ____________________________________________
3. ____________________________________________

---

## Audit Summary

| Area | Status | Notes |
|------|--------|-------|
| File inventory | ✅ / ⚠️ / ❌ | |
| Accuracy | ✅ / ⚠️ / ❌ | |
| Effectiveness | ✅ / ⚠️ / ❌ | |
| Budget | ✅ / ⚠️ / ❌ | |
| Consistency | ✅ / ⚠️ / ❌ | |
| Security | ✅ / ⚠️ / ❌ | |
| Team feedback | ✅ / ⚠️ / ❌ | |

**Auditor**: _______________
**Date**: _______________
**Next audit due**: _______________

---

*Next: [Migration Guide](migration-guide.md) →*
