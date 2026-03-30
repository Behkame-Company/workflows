# Maintaining Instructions Over Time

> How to keep instruction files accurate, effective, and aligned with your evolving codebase.

---

## Why Maintenance Matters

Instructions **decay** over time:
- Libraries get upgraded (React 18 → 19)
- Conventions change (migrate from CSS Modules to Tailwind)
- Team grows (new patterns emerge)
- Code structure evolves (new directories, renamed modules)
- Best practices shift (industry changes)

Stale instructions are worse than no instructions — they actively mislead Copilot.

---

## Maintenance Cadence

| Activity | Frequency | Owner |
|----------|-----------|-------|
| Quick review (still accurate?) | Monthly | Any developer |
| Full audit (effectiveness test) | Quarterly | Tech lead |
| Stack version update | After each upgrade | Developer doing the upgrade |
| Convention change | When decision is made | Decision maker |
| Structure review | After major refactoring | Refactoring developer |

---

## Monthly Quick Review

Spend 5 minutes checking:

- [ ] Commands still work? (`npm test` still the right command?)
- [ ] Stack versions current? (Still React 18, or now 19?)
- [ ] Directory structure matches? (Any new/renamed/deleted folders?)
- [ ] No contradictions? (Did a recent decision change a rule?)
- [ ] No dead references? (Do referenced files still exist?)

### Template: Monthly Review PR

```markdown
# Monthly Copilot Instructions Review

## Checked
- [x] Commands current
- [x] Stack versions accurate
- [ ] Structure updated — `src/hooks/` was renamed to `src/composables/`
- [x] No contradictions
- [x] References valid

## Changes Made
- Updated directory structure to reflect `hooks → composables` rename
- Updated React version from 18 to 19 in project section
```

---

## Quarterly Full Audit

### Step 1: Effectiveness Test

For each instruction, test if Copilot actually follows it:

1. Open a relevant file
2. Ask Copilot to do something the instruction addresses
3. Grade: Does Copilot comply?

| Instruction | Test | Result | Action |
|-------------|------|--------|--------|
| "Use named exports" | "Create a utility function" | ✅ Named export | Keep |
| "Validate with Zod" | "Create an API endpoint" | ❌ No Zod used | Rewrite — too buried in file |
| "Use the logger" | "Add error handling" | ⚠️ Sometimes | Move to top of file |

### Step 2: Relevance Review

Ask for each instruction:
- Is this still our practice?
- Is Copilot the right place to enforce this? (Maybe the linter handles it now?)
- Does this actually change Copilot's behavior? (If Copilot already does it, remove it)

### Step 3: Budget Check

- Count total words across all loaded files
- Remove lowest-value instructions if over budget
- Consolidate duplicated rules

---

## Triggers for Immediate Updates

Update instructions **immediately** when:

| Trigger | Example |
|---------|---------|
| Framework upgrade | React 18 → 19, Next.js 14 → 15 |
| Build tool change | Webpack → Vite, Jest → Vitest |
| Convention decision | Team votes to switch from CSS Modules to Tailwind |
| Major refactor | src/lib/ split into src/utils/ + src/services/ |
| New tool adoption | Adding Prisma, switching to pnpm |
| Security incident | "Never use library X" after vulnerability |
| Team feedback | "Copilot keeps doing Y wrong" |

### Process: Convention Change

```
1. Team makes decision (e.g., "We're switching to Tailwind")
2. Update copilot-instructions.md or relevant .instructions.md
3. If explicit override needed: "Use Tailwind CSS (not CSS Modules — we are migrating)"
4. Commit instruction changes BEFORE the code changes
5. AI tools immediately start using the new convention
```

---

## Version Control for Instructions

### Commit Message Convention

```
chore(copilot): update instructions for React 19 migration
chore(copilot): add database migration safety rules
chore(copilot): remove stale directory structure reference
chore(copilot): quarterly audit — refresh all instruction files
```

### PR Review for Instructions

Instruction file changes should be reviewed like code:
- Does the instruction make sense?
- Is it specific and actionable?
- Does it contradict existing rules?
- Is it within budget?

### Blame History

Use `git blame` on instruction files to understand when and why rules were added. Remove rules whose original context no longer exists.

---

## Measuring Drift

### Signs Your Instructions Have Drifted

| Symptom | Cause |
|---------|-------|
| Copilot suggests old patterns | Instructions reference deprecated approaches |
| "Copilot is ignoring my instructions" | Instructions are stale or too wordy |
| New team members confused by AI output | Instructions don't match current practices |
| AI creates code in nonexistent directories | Structure section is outdated |
| Build fails after AI changes | Build commands have changed |

### Drift Prevention

1. **Tie instruction updates to code changes** — if you rename a directory, update instructions in the same PR
2. **Include instructions in onboarding** — new developers should review instruction files
3. **Quarterly audit reminders** — set a recurring calendar event
4. **CODEOWNERS** — assign ownership of `.github/copilot-instructions.md`

---

## CODEOWNERS for Instructions

Add to `.github/CODEOWNERS`:

```
# Copilot instruction files — require tech lead review
.github/copilot-instructions.md @team-lead @platform-team
.github/instructions/             @team-lead
.github/prompts/                   @team-lead
AGENTS.md                          @team-lead
```

This ensures all instruction changes go through a designated reviewer.

---

## Deprecation Workflow

When removing or changing a rule:

```markdown
# Step 1: Mark as deprecated (keep for one sprint)
- ~~Use CSS Modules for styling~~ DEPRECATED: migrating to Tailwind

# Step 2: Add new rule alongside
- Use Tailwind CSS for all new components
- Legacy CSS Modules in `src/legacy/` — do not modify, only read

# Step 3: Remove deprecated rule after migration complete
- Use Tailwind CSS for all styling
```
