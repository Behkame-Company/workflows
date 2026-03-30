# Common SDD Mistakes

> The top 10 mistakes teams make when adopting Spec-Driven Development.

---

## Mistake #1: Skipping the Spec for "Simple" Features

That "simple button" ends up with 12 edge cases and 3 accessibility issues. Even simple features benefit from a 5-minute spec — write 3 acceptance criteria and 2 edge cases.

---

## Mistake #2: Specs as Bureaucracy, Not Communication

When specs are checkbox exercises, they're vague and nobody reads them. Write specs so a new team member could implement the feature — if they're boring to write, they're not specific enough.

---

## Mistake #3: Specifying Implementation Instead of Intent

Specs that dictate implementation (e.g., "Use Redis sorted sets") constrain better solutions and mix concerns. Spec the *what* and *why* — save *how* for the Plan phase. Example: ✅ "Leaderboard shows top 100 users by score, updated within 5 seconds."

---

## Mistake #4: No "Out of Scope" Section

Without explicit scope boundaries, stakeholders assume features are included and AI agents over-implement. Every spec needs an "Out of Scope" section listing 3–5 things that might seem in-scope but aren't.

---

## Mistake #5: Write-Once Specs

Specs never updated after approval become fiction. Update specs when you learn new information, track changes with version numbers, and make spec updates part of the implementation PR.

---

## Mistake #6: AI-Generated Specs Without Review

AI writes fluent, convincing specs that may be completely wrong — hallucinated requirements, missing business context, generic patterns. AI can *draft* specs; humans must review, refine, and approve.

---

## Mistake #7: Spec Bloat

Over-specified features waste time and overflow AI context windows. A medium feature needs 50–100 lines of spec, not 300. If the spec is longer than the code will be, it's too long.

---

## Mistake #8: No Quality Gates

Without quality gates between phases, errors cascade — vague spec → wrong plan → broken tasks → incorrect code. Add mandatory review checkpoints between each phase, even if it's a 5-minute self-check.

---

## Mistake #9: Spec Per-System Instead of Per-Feature

Giant system-level specs are unwieldy and outdated before they're finished. Write per-feature specs that are independently implementable: "User Registration Spec" (2 pages) instead of "User Management System Spec" (50 pages).

---

## Mistake #10: Ignoring the Constitution

Without a project constitution, every spec duplicates or inconsistently defines project-wide constraints. Create a constitution once, reference it from every spec, update it when standards change.

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
