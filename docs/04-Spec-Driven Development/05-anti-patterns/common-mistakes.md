# Common SDD Mistakes

> The top 10 mistakes teams make when adopting Spec-Driven Development.

---

## Mistake #1: Skipping the Spec for "Simple" Features

> "It's just a button — we don't need a spec."

Three months later, that button has 12 edge cases, 3 accessibility issues, and 2 security bugs — none of which were considered upfront.

**The Fix**: Even "simple" features benefit from a 5-minute spec. Write 3 acceptance criteria and 2 edge cases. If it really is simple, this takes 5 minutes. If it's not, you just saved yourself hours.

---

## Mistake #2: Specs as Bureaucracy, Not Communication

> "We write specs because the process requires it."

When specs are checkbox exercises rather than genuine communication tools, they're vague, copy-pasted, and nobody reads them.

**The Fix**: Treat specs as the primary conversation artifact. Write them so a new team member could implement the feature. If your specs are boring to write, they're probably not specific enough.

---

## Mistake #3: Specifying Implementation Instead of Intent

> "Use a PostgreSQL JSONB column with GIN index for the search feature."

When specs dictate implementation, they:
- Constrain better solutions the AI might find
- Become stale when technology changes
- Mix concerns (what vs. how)

**The Fix**: Spec the *what* and *why*. Save *how* for the Plan phase.

```markdown
❌  Spec: "Use Redis sorted sets for the leaderboard"
✅  Spec: "Leaderboard shows top 100 users by score, updated within 5 seconds"
```

---

## Mistake #4: No "Out of Scope" Section

> "I assumed notifications were included."

Without explicit scope boundaries, stakeholders assume features are included, developers build unrequested functionality, and AI agents over-implement.

**The Fix**: Every spec has an "Out of Scope" section listing 3–5 things that might seem in-scope but aren't.

---

## Mistake #5: Write-Once Specs

> "The spec was approved in January. We're still implementing in March."

Specs that are never updated after approval become fiction. Requirements change, edge cases are discovered, and the spec becomes misleading.

**The Fix**: Update specs when you learn new information. Track changes with version numbers. Make spec updates part of the implementation PR.

---

## Mistake #6: AI-Generated Specs Without Review

> "Let Copilot write the spec. It knows what we need."

AI agents write fluent, convincing specs that may be completely wrong. They hallucinate requirements, miss your specific business context, and apply generic patterns.

**The Fix**: AI can draft specs. Humans must review, refine, and approve them. Use AI as a starting point, not the final word.

```
AI drafts spec (60% correct)
  → Human reviews (catches gaps)
  → Human adds domain knowledge (90% correct)
  → Team review (98% correct)
  → Approved ✅
```

---

## Mistake #7: Spec Bloat

> "Let me add one more section... and another edge case... and an appendix..."

Over-specified features take longer to write, longer to review, and longer to implement. The AI agent's context window fills up with spec text, leaving less room for reasoning.

**The Fix**: Right-size specs. A medium feature needs 50–100 lines, not 300. If the spec is longer than the code will be, it's too long.

---

## Mistake #8: No Quality Gates

> "We wrote the spec, so we can skip the review."

Without quality gates between phases, errors cascade. A vague spec leads to a wrong plan leads to broken tasks leads to incorrect code.

**The Fix**: Mandatory review checkpoints between each phase, even if "review" means a 5-minute self-check with a checklist.

---

## Mistake #9: Spec Per-System Instead of Per-Feature

> "Let's write one big spec for the entire user management system."

Giant specs that cover entire systems are unwieldy, unreviewed, and unmaintainable. They take weeks to write and are outdated before they're finished.

**The Fix**: Write specs per-feature, not per-system. Each spec should be independently implementable and completable.

```
❌  "User Management System Spec" (50 pages)
✅  "User Registration Spec" (2 pages)
✅  "User Profile Spec" (2 pages)
✅  "User Permissions Spec" (3 pages)
```

---

## Mistake #10: Ignoring the Constitution

> "Every spec reinvents our coding standards."

Without a project constitution, every spec duplicates project-wide constraints, or worse, defines them inconsistently.

**The Fix**: Create a constitution once. Reference it from every spec. Update it when project-wide standards change.

---

## Impact Summary

| Mistake | Impact | Prevention Cost |
|---------|--------|----------------|
| Skipping specs for "simple" features | Bugs, rework | 5 min per feature |
| Specs as bureaucracy | Low quality specs | Team training |
| Specifying implementation | Constrained solutions | Spec review |
| No "Out of Scope" | Feature creep | 2 min per spec |
| Write-once specs | Spec-code drift | Ongoing updates |
| AI specs without review | Wrong requirements | Review checkpoint |
| Spec bloat | Wasted time, context overflow | Size guidelines |
| No quality gates | Cascading errors | Checklists |
| System-level specs | Unwieldy documents | Feature decomposition |
| Ignoring constitution | Inconsistent standards | Constitution file |

---

*Next: [Waterfall in Disguise →](waterfall-in-disguise.md)*
