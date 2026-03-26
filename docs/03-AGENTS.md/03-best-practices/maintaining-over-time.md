# Maintaining AGENTS.md Over Time

> Lifecycle management, audit cadence, and preventing instruction drift.

---

## The Maintenance Problem

AGENTS.md files rot. Projects evolve — frameworks update, patterns change, directories move — but AGENTS.md stays frozen at the day it was written.

**Stale AGENTS.md is worse than no AGENTS.md.** It actively misleads agents:
- Points to directories that no longer exist
- References deprecated APIs or tools
- Specifies wrong test commands
- Contradicts current conventions

---

## The AGENTS.md Lifecycle

```
┌─────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Create   │ ──→ │ Iterate  │ ──→ │ Audit    │ ──→ │ Archive/ │
│          │     │          │     │          │     │ Rewrite  │
│ 30 lines │     │ Observe  │     │ Quarterly│     │ When     │
│ minimum  │     │ + refine │     │ review   │     │ needed   │
└─────────┘     └──────────┘     └──────────┘     └──────────┘
     ↑                                                  │
     └──────────────────────────────────────────────────┘
```

### Phase 1: Create (Day 1)

Start minimal:
- Setup commands
- Stack and versions
- 3-5 conventions
- 3-5 boundaries

**Target: 30-50 lines. Don't try to be comprehensive on day one.**

### Phase 2: Iterate (Ongoing)

After every agent task, ask:
1. Did the agent follow the conventions? → If not, add/clarify the rule.
2. Did the agent break something? → Add a boundary.
3. Did the agent use the wrong command? → Fix the setup section.
4. Did the agent produce great output? → No change needed.

**Growth rate: ~2-5 lines per week, max.**

### Phase 3: Audit (Quarterly)

Schedule a quarterly review (see [Audit Checklist](../08-practical/audit-checklist.md)):
1. Run every command in the setup section — do they all work?
2. Check every directory in the structure section — do they all exist?
3. Review every convention — is it still current?
4. Review every boundary — is it still needed?
5. Check file size — has it grown past 150 lines?

### Phase 4: Rewrite (When Needed)

Major triggers for a full rewrite:
- Framework migration (Next.js 14 → 15, CRA → Vite)
- Language change (JavaScript → TypeScript)
- Architecture change (monolith → monorepo)
- Team structure change (solo → team, team → enterprise)

---

## Drift Detection Strategies

### Strategy 1: CI Validation

Add a CI check that validates AGENTS.md references:

```yaml
# .github/workflows/agents-md-check.yml
name: AGENTS.md Validation
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check referenced commands
        run: |
          # Verify package.json scripts match AGENTS.md
          for cmd in $(grep -oP '`(npm|pnpm|yarn) \w+`' AGENTS.md | tr -d '`'); do
            if ! grep -q "${cmd##* }" package.json; then
              echo "WARNING: $cmd not found in package.json"
            fi
          done
      - name: Check referenced directories
        run: |
          for dir in $(grep -oP '[a-zA-Z]+/' AGENTS.md | sort -u); do
            if [ ! -d "$dir" ]; then
              echo "WARNING: Directory $dir referenced in AGENTS.md does not exist"
            fi
          done
```

### Strategy 2: CODEOWNERS for AGENTS.md

```
# .github/CODEOWNERS
AGENTS.md @tech-lead @senior-devs
```

All changes to AGENTS.md require review by designated owners.

### Strategy 3: PR Template Reminder

```markdown
# .github/pull_request_template.md
## Checklist
- [ ] If changing conventions or project structure, update AGENTS.md
- [ ] If adding/changing build/test commands, update AGENTS.md setup section
```

### Strategy 4: Dependabot / Renovate Integration

When dependencies update, flag AGENTS.md for review:

```yaml
# Add to PR template for dependency updates
## AGENTS.md Review Needed?
- Does this update change any build/test commands?
- Does this update introduce new patterns or deprecate old ones?
- Does the version number in AGENTS.md need updating?
```

---

## Common Drift Scenarios

| What Changed | AGENTS.md Impact |
|-------------|-----------------|
| New framework version | Update Stack section with version |
| Package manager change | Update ALL commands in Setup |
| Directory restructure | Update Structure section |
| New convention adopted | Add to Conventions |
| Convention abandoned | Remove from Conventions |
| New CI/CD pipeline | Update workflow references |
| Team member preference | Only update if it's now team standard |

---

## Version Control Best Practices

### Treat AGENTS.md Like Code

```bash
# Good: Specific, explanatory commit messages
git commit -m "docs(AGENTS.md): update test command to vitest"
git commit -m "docs(AGENTS.md): add boundary for generated/ directory"
git commit -m "docs(AGENTS.md): remove stale webpack references"

# Bad: Generic commit messages
git commit -m "update AGENTS.md"
git commit -m "fix docs"
```

### Review AGENTS.md Changes in PRs

AGENTS.md changes should be reviewed with the same rigor as configuration changes:
- Does the new rule make sense?
- Is it specific enough for agents to follow?
- Does it conflict with existing rules?
- Is it tested?

---

## The Update Trigger Matrix

| Event | Action |
|-------|--------|
| Agent produces wrong code | Add/clarify convention rule |
| Agent touches protected file | Add boundary rule |
| Agent runs wrong command | Fix setup section |
| Framework major update | Update stack + commands |
| New tool added to project | Add to stack + setup |
| Quarterly calendar reminder | Full audit (see checklist) |
| New team member starts | Review with fresh eyes |
| Agent consistently ignores rule | Rewrite rule with stronger phrasing |

---

*Next: [Common Mistakes](../04-anti-patterns/common-mistakes.md) →*
