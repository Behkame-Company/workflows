# Resource & Prompt Design

> Design resources that provide the right context and prompts that guide consistent AI behavior.

---

## Designing Resources

### Principle: Right-Size Your Data

The AI's context window is limited. Don't dump everything — provide what's needed:

```
❌ resource: "database://full-dump"     → 50,000 rows of raw data
✅ resource: "database://schema"         → Table definitions only
✅ resource: "database://recent-errors"  → Last 10 error entries
```

### Resource Design Checklist

1. **Is this read-only?** If the AI needs to modify data, use a tool instead
2. **Is the size bounded?** Implement truncation or pagination
3. **Is the URI descriptive?** The AI uses URIs to understand content
4. **Is the MIME type correct?** Helps the AI parse the format
5. **Does it change often?** Consider supporting subscriptions

### Example: Well-Designed Resources

```python
@mcp.resource("project://dependencies")
async def get_dependencies() -> str:
    """Current project dependencies with versions."""
    with open("package.json") as f:
        pkg = json.load(f)
    deps = pkg.get("dependencies", {})
    return "\n".join(f"  {k}: {v}" for k, v in deps.items())

@mcp.resource("project://architecture")
async def get_architecture() -> str:
    """High-level project architecture overview."""
    return """
    Architecture: Next.js Full-Stack
    Frontend: React + TypeScript (src/app/)
    Backend: API Routes (src/app/api/)
    Database: PostgreSQL via Prisma (prisma/schema.prisma)
    Auth: NextAuth.js with JWT
    """
```

---

## Designing Prompts

### Principle: Prompts Are Reusable Recipes

Prompts codify **team workflows** so every developer (and every AI session) follows the same process.

### Anatomy of a Great Prompt

1. **Role assignment** — Tell the AI who it is
2. **Task context** — Embed relevant resources
3. **Output structure** — Define the expected format
4. **Constraints** — What to avoid or prioritize

### Example: Code Review Prompt

```python
@mcp.prompt
async def code_review(pr_number: str, focus: str = "all") -> list:
    """Perform a structured code review on a pull request."""
    return [
        {
            "role": "system",
            "content": f"""You are a senior engineer performing a code review.
Focus area: {focus}

Review structure:
1. **Summary**: What does this PR do? (2-3 sentences)
2. **Critical Issues**: Bugs, security flaws, data loss risks
3. **Improvements**: Performance, readability, maintainability
4. **Positive Notes**: What's done well
5. **Verdict**: Approve / Request Changes / Needs Discussion

Rules:
- Only flag issues that genuinely matter
- Never comment on formatting (that's what linters are for)
- Include line references where possible
- Be specific with suggestions (show code, not vague advice)"""
        },
        {
            "role": "user",
            "content": {
                "type": "resource",
                "resource": { "uri": f"github://pulls/{pr_number}/diff" }
            }
        }
    ]
```

### Example: Bug Report Prompt

```python
@mcp.prompt
async def bug_report(description: str) -> str:
    """Generate a structured bug report from a description."""
    return f"""Create a structured bug report from this description:

{description}

Format:
## Bug Report
**Title**: [Concise, specific title]
**Severity**: [Critical/High/Medium/Low]
**Steps to Reproduce**: [Numbered list]
**Expected Behavior**: [What should happen]
**Actual Behavior**: [What actually happens]
**Environment**: [OS, browser, version if known]
**Possible Cause**: [Your analysis if any]
**Suggested Fix**: [If you have one]"""
```

---

## Resources + Prompts Together

The most powerful patterns **combine resources and prompts**:

```python
@mcp.prompt
async def migration_review(migration_file: str) -> list:
    """Review a database migration for safety."""
    return [
        {
            "role": "system",
            "content": "You are a database expert reviewing a migration for safety."
        },
        {
            "role": "user",
            "content": {
                "type": "resource",
                "resource": { "uri": f"file://{migration_file}" }
            }
        },
        {
            "role": "user",
            "content": {
                "type": "resource",
                "resource": { "uri": "database://schema" }
            }
        },
        {
            "role": "user",
            "content": "Review this migration against the current schema. Check for: data loss, breaking changes, missing indexes, and rollback safety."
        }
    ]
```

---

## Key Takeaways

1. **Resources**: Right-size data, use descriptive URIs, bound the output size
2. **Prompts**: Define role + context + structure + constraints
3. **Combine them**: Prompts that reference resources create powerful workflows
4. **Version your prompts**: As team practices evolve, update templates
5. **Test with real AI**: Verify the AI produces the expected output format

---

## Next Steps

- 🔗 [Testing & Debugging](testing-and-debugging.md) — Validate your implementations
- 🔗 [Server Design Principles](../06-best-practices/server-design-principles.md) — Overarching design rules
