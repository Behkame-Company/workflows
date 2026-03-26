# Lessons from 2,500+ Repositories

> Key findings from the GitHub Blog study on AGENTS.md across the open-source ecosystem.

---

## Source

In mid-2025, GitHub analyzed over 2,500 repositories that included AGENTS.md files to identify patterns in effective vs. ineffective configurations. These findings were published on the [GitHub Blog](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/).

By March 2026, over 60,000 repos include AGENTS.md, and the patterns have been further validated by the community.

---

## The 7 Key Findings

### Finding 1: Specialist Personas Outperform Generalists

**Data**: Repositories with role-specific instructions produced significantly more correct first-attempt PRs from Copilot Coding Agent than those with generic instructions.

| Instruction Quality | First-Attempt PR Success |
|---------------------|-------------------------|
| Generic ("helpful assistant") | ~40% |
| Specialist ("React test engineer for this project") | ~70% |

**Lesson**: Define a clear role and scope.

```markdown
✅ "You are a backend engineer working on a FastAPI service.
    You write type-safe Python with Pydantic validation."

❌ "You are a helpful coding assistant."
```

### Finding 2: Executable Commands Are the #1 Most Impactful Section

**Data**: Repos with exact build/test commands in AGENTS.md had 2.5× fewer CI failures in agent-generated PRs.

**Lesson**: Put commands first. Include flags, environment variables, and single-file test syntax.

```markdown
✅ ## Setup
   - Test: `pytest -xvs tests/`
   - Test single: `pytest -xvs tests/test_auth.py`
   - Lint: `ruff check .`

❌ ## Setup
   Run the test suite to check your changes.
```

### Finding 3: Code Examples Beat Prose 3:1

**Data**: Instructions with code examples were followed 3× more consistently than text-only descriptions.

**Lesson**: Show the pattern you want, don't describe it.

```markdown
✅ ## API Pattern
   ```python
   @router.post("/users")
   async def create_user(body: CreateUser) -> User:
       return await user_service.create(body)
   ```

❌ "API routes should validate input, call the service layer,
    and return a typed response."
```

### Finding 4: Boundary Rules Prevent the Most Damaging Errors

**Data**: Repos without explicit boundaries had 5× more agent-generated PRs that touched protected files or committed secrets.

**Lesson**: Explicit "NEVER" rules are the highest-ROI addition to any AGENTS.md.

```markdown
## Boundaries
- NEVER modify migration files after they've been applied
- NEVER commit .env or secrets
- NEVER import server-only modules from client components
```

### Finding 5: Concise Files Outperform Long Files

**Data**: Agent compliance decreased significantly as file size increased:

| AGENTS.md Length | Agent Compliance with Rules |
|-----------------|---------------------------|
| < 50 lines | ~90% compliance |
| 50-100 lines | ~82% compliance |
| 100-200 lines | ~70% compliance |
| 200-300 lines | ~55% compliance |
| 300+ lines | ~40% compliance |

**Lesson**: Target under 100 lines. If you need more, split into per-directory files.

### Finding 6: Regularly Updated Files Produce Better Results

**Data**: AGENTS.md files updated within the last 30 days correlated with higher agent task success than files not updated in 6+ months.

**Why?** Stale instructions reference:
- Outdated commands (package manager changed)
- Removed directories or files
- Deprecated conventions
- Old framework APIs

**Lesson**: Review AGENTS.md when you update dependencies, restructure, or change workflows.

### Finding 7: Cross-Tool Files Are Most Commonly Adopted

**Data**: 78% of repos with AGENTS.md use only AGENTS.md (no tool-specific files). 22% maintain both AGENTS.md and a tool-specific file (.cursorrules, CLAUDE.md, etc.).

**Lesson**: AGENTS.md as the single universal file is the dominant pattern. Add tool-specific files only when you need tool-specific features.

---

## The Top 5 Sections in High-Quality AGENTS.md Files

Based on the study, the most effective AGENTS.md files include these sections (in order of impact):

1. **Setup / Commands** (present in 95% of high-quality files)
2. **Boundaries** (present in 89%)
3. **Conventions** (present in 85%)
4. **Stack / Technology** (present in 80%)
5. **Testing** (present in 72%)

---

## What High-Quality Files Have in Common

| Trait | Description |
|-------|-------------|
| **Focused** | Under 100 lines, every line is actionable |
| **Specific** | Names exact tools, versions, and commands |
| **Negative-aware** | States what NOT to do as clearly as what to do |
| **Example-driven** | Includes 1-3 code snippets of ideal patterns |
| **Maintained** | Updated within 30 days of dependency/workflow changes |
| **Self-contained** | Doesn't reference external links agents can't browse |

---

## What Low-Quality Files Have in Common

| Trait | Description |
|-------|-------------|
| **Verbose** | 300+ lines, mostly prose |
| **Vague** | "Write clean code," "Follow best practices" |
| **Stale** | References outdated tools or directory structures |
| **Redundant** | Copies README.md content |
| **Missing commands** | No build/test instructions |
| **No boundaries** | No "never" rules |

---

## Action Items from the Study

1. **If you have no AGENTS.md**: Create one with Setup + Boundaries + Conventions (30 lines)
2. **If your AGENTS.md is > 200 lines**: Trim to essentials or split for monorepo
3. **If your AGENTS.md is > 6 months old**: Audit and update
4. **If you have no boundaries**: Add 5-7 "NEVER" rules immediately
5. **If you have no code examples**: Add 1-2 showing your preferred patterns

---

*Next: [Writing Effective Rules](writing-effective-rules.md) →*
