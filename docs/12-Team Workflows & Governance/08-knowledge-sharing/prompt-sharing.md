# Prompt Sharing & Collaboration

> Build systems for sharing effective prompts across your team so that one developer's breakthrough becomes everyone's productivity gain.

---

## Why Share Prompts?

Every developer on your team is independently discovering what works with AI tools. Without a sharing system, those discoveries are:

- **Lost** when someone switches projects or leaves the team
- **Siloed** in individual workflows nobody else sees
- **Inconsistent** — ten developers writing ten different prompts for the same task
- **Unreproducible** — "it worked for me" without documentation of context

Prompt sharing transforms individual experimentation into collective intelligence.

### The Compound Effect

When a team of 8 developers each discovers 2 effective prompts per month, that's 192 prompts per year. Without sharing, each developer has access to 24. With sharing, each has access to 192 — an **8x multiplier** on AI effectiveness.

✅ **Good mindset:** "I found a great prompt for generating API tests — let me share it with the team"
❌ **Bad mindset:** "I have my own prompt tricks and they give me an edge over my teammates"

---

## Sharing Mechanisms

### 1. `.prompt.md` Files in Repository

The most powerful mechanism: prompts that live alongside your code and are discoverable by AI tools.

#### How It Works

Place `.prompt.md` files in relevant directories. AI tools like GitHub Copilot can discover and use these as context.

```
repo/
├── .github/
│   ├── copilot-instructions.md
│   └── prompts/
│       ├── generate-api-test.prompt.md
│       ├── review-security.prompt.md
│       └── refactor-component.prompt.md
├── src/
│   ├── api/
│   │   └── generate-endpoint.prompt.md
│   ├── components/
│   │   └── create-component.prompt.md
│   └── database/
│       └── write-migration.prompt.md
```

#### `.prompt.md` File Format

```markdown
---
title: Generate API Integration Test
category: testing
author: "@sarah"
tested: 2025-01-10
tools: [copilot-chat, copilot-cli]
effectiveness: 4/5
---

# Generate API Integration Test

Write a comprehensive integration test for the specified API endpoint.

## Context
- Use the test framework configured in this project (Jest + Supertest)
- Import test helpers from `src/test/helpers.ts`
- Follow the Arrange-Act-Assert pattern
- Use factories from `src/test/factories/` for test data

## Requirements
- Test the happy path (200 response with valid data)
- Test authentication failures (401 without token)
- Test authorization failures (403 with insufficient permissions)
- Test validation errors (400 with malformed input)
- Test not-found scenarios (404 with invalid IDs)
- Verify response shape matches the OpenAPI schema

## Output Format
- One `describe` block per endpoint
- Nested `describe` blocks for each scenario category
- Clear test names: `it('should return 200 with valid user data')`
```

✅ **Good `.prompt.md`:** Specific, includes context about the project's tooling, tested and dated
❌ **Bad `.prompt.md`:** Generic prompt that could apply to any project without customization

---

### 2. Shared Skills Repository

A dedicated repository for reusable AI skills and prompts that work across projects.

#### Repository Structure

```
org/ai-skills/
├── README.md                    # How to contribute and use
├── CONTRIBUTING.md              # Contribution guidelines
├── skills/
│   ├── testing/
│   │   ├── unit-test.prompt.md
│   │   ├── integration-test.prompt.md
│   │   └── e2e-test.prompt.md
│   ├── code-review/
│   │   ├── security-review.prompt.md
│   │   ├── performance-review.prompt.md
│   │   └── accessibility-review.prompt.md
│   ├── documentation/
│   │   ├── api-docs.prompt.md
│   │   ├── readme-generator.prompt.md
│   │   └── changelog-entry.prompt.md
│   └── refactoring/
│       ├── extract-function.prompt.md
│       ├── simplify-conditional.prompt.md
│       └── modernize-syntax.prompt.md
├── templates/
│   └── prompt-template.prompt.md
└── archive/
    └── deprecated/              # Old prompts kept for reference
```

#### How Teams Consume Shared Skills

```bash
# Option 1: Git submodule
git submodule add git@github.com:org/ai-skills.git .github/shared-skills

# Option 2: Copy specific prompts
curl -o .github/prompts/security-review.prompt.md \
  https://raw.githubusercontent.com/org/ai-skills/main/skills/code-review/security-review.prompt.md

# Option 3: Symlink (monorepo)
ln -s ../../shared/ai-skills/testing .github/prompts/testing
```

✅ **Good shared repo:** Well-organized, easy to contribute, templates for consistency
❌ **Bad shared repo:** Dump of random prompts with no organization or contribution process

---

### 3. Team Slack Channel for Prompt Tips

A low-friction channel for real-time prompt sharing and discussion.

#### Channel Setup: `#ai-prompts`

```markdown
## Channel Purpose
Share effective prompts, ask for prompt help, and discuss
AI-assisted development techniques.

## Channel Norms
- 🎯 Share prompts that worked well (include the context!)
- ❓ Ask for help crafting prompts for specific tasks
- 👍 React to prompts you've tried and found useful
- 📝 If a prompt gets 5+ reactions, add it to the prompt library
- 🔄 Weekly: Top-reacted prompts get promoted to the shared repo

## Post Format
**Task:** [What you were trying to do]
**Prompt:** [The prompt you used]
**Result:** [What happened — screenshot or summary]
**Rating:** ⭐⭐⭐⭐ (4/5)
**Tip:** [Any nuance that matters]
```

#### Slack Workflow Automation

Set up a Slack workflow that:
1. Collects prompt submissions via a form
2. Posts them in a standardized format
3. After 5+ 👍 reactions, automatically creates a GitHub issue to add to the shared repo

✅ **Good channel culture:** Regular sharing with context, constructive feedback, celebration of discoveries
❌ **Bad channel culture:** Only complaints about AI, no sharing of successes, thread ghost town

---

### 4. Monthly Prompt Showcase

A regular meeting where team members demo their most effective prompts.

#### Showcase Format (30 minutes)

```markdown
## Monthly Prompt Showcase — Agenda

### Opening (2 min)
- Quick stats: prompts shared this month, most popular

### Demos (20 min, 4 slots × 5 min each)
Each presenter:
1. **Problem** — What task were you trying to accomplish?
2. **Prompt** — Show the exact prompt (live demo preferred)
3. **Result** — What did the AI produce?
4. **Iteration** — How did you refine it?
5. **Takeaway** — What principle makes this prompt effective?

### Discussion (5 min)
- What patterns do we see across today's prompts?
- Any prompts we should standardize?

### Closing (3 min)
- Vote for "Prompt of the Month"
- Assign someone to add top prompts to the shared library
```

#### Showcase Best Practices

| Do | Don't |
|----|-------|
| Live demo when possible | Read from slides |
| Show failures and iteration | Only show the final polished version |
| Explain *why* it works | Just show the prompt without context |
| Encourage questions | Rush through to fit more demos |
| Record for async viewing | Assume everyone can attend live |

✅ **Good showcase:** Developer shows 3 prompt iterations, explains why version 3 worked, and others adapt it live
❌ **Bad showcase:** PowerPoint presentation about prompt engineering theory with no hands-on examples

---

### 5. Prompt Rating System

A mechanism for the team to signal which prompts are actually useful.

#### Rating Dimensions

| Dimension | Scale | Description |
|-----------|-------|-------------|
| Effectiveness | 1-5 ⭐ | How well does it produce the desired output? |
| Consistency | 1-5 🎯 | Does it work reliably across different inputs? |
| Adaptability | 1-5 🔧 | How easy is it to modify for similar tasks? |
| Time saved | 1-5 ⏱️ | How much faster is this vs. doing it manually? |

#### Rating in Practice

Add a rating block to each `.prompt.md`:

```markdown
## Community Ratings

| Reviewer | Effectiveness | Consistency | Adaptability | Time Saved | Notes |
|----------|:---:|:---:|:---:|:---:|-------|
| @alice | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Works great for REST APIs |
| @bob | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Needed tweaks for GraphQL |
| @carol | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | Best for CRUD endpoints |

**Average:** 4.3/5 | **Times Used:** 23 | **Last Rated:** 2025-01-12
```

✅ **Good rating system:** Multiple dimensions, notes with context, tracks usage over time
❌ **Bad rating system:** Simple thumbs up/down with no context about what was tested

---

## Prompt Contribution Process

### The Prompt Pipeline

```
┌─────────────┐     ┌─────────────┐     ┌──────────────┐
│  Discovery   │────▶│   Testing    │────▶│ Documentation │
│              │     │              │     │               │
│ Developer    │     │ Try on 3+    │     │ Fill out      │
│ finds an     │     │ different    │     │ template with │
│ effective    │     │ scenarios    │     │ examples      │
│ prompt       │     │              │     │               │
└─────────────┘     └─────────────┘     └──────────────┘
                                               │
                                               ▼
┌─────────────┐     ┌─────────────┐     ┌──────────────┐
│   Merged     │◀────│   Review     │◀────│  Submit PR   │
│              │     │              │     │               │
│ Added to     │     │ Another dev  │     │ PR to shared  │
│ shared       │     │ tests and    │     │ prompt repo   │
│ library      │     │ reviews      │     │ or docs/      │
└─────────────┘     └─────────────┘     └──────────────┘
```

### Step 1: Discovery

You've found a prompt that works well. Before sharing:

- [ ] Used it successfully at least 3 times
- [ ] Tested with different inputs (not just one lucky case)
- [ ] Understood *why* it works (not just *that* it works)

### Step 2: Testing on Multiple Scenarios

```markdown
## Testing Checklist

- [ ] Works with small inputs (single function)
- [ ] Works with medium inputs (single file)
- [ ] Works with large inputs (multiple files)
- [ ] Works with different programming languages (if applicable)
- [ ] Produces consistent output format
- [ ] Gracefully handles edge cases
- [ ] Tested by at least one other developer
```

### Step 3: Documentation

Use the prompt contribution template (below) to document your prompt thoroughly.

### Step 4: Submit PR

```bash
# Fork or branch the shared prompt repo
git checkout -b prompt/generate-api-test

# Add your prompt file
cp my-prompt.prompt.md skills/testing/generate-api-test.prompt.md

# Commit and push
git add .
git commit -m "Add API integration test generation prompt

Tested across 3 projects with REST and GraphQL endpoints.
Average effectiveness: 4/5.

Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>"

git push origin prompt/generate-api-test
```

### Step 5: Review

The reviewer should:

- [ ] Read the prompt and understand its intent
- [ ] Actually test the prompt in their own environment
- [ ] Verify the examples match the actual output
- [ ] Check for security concerns (does it handle sensitive data safely?)
- [ ] Suggest improvements if applicable

### Step 6: Merge and Announce

After merge:
- Post in `#ai-prompts` Slack channel
- Include in next monthly showcase
- Update the prompt library index

✅ **Good contribution:** Thorough testing, clear documentation, real examples from multiple scenarios
❌ **Bad contribution:** "This prompt worked once for me, here it is" with no testing or examples

---

## Prompt Attribution

Credit the people who create and improve prompts.

### Attribution Practices

```markdown
## Prompt Metadata
- **Original author:** @sarah (2025-01-05)
- **Contributors:** @mike (added GraphQL support), @alex (improved error handling)
- **Based on:** [Link to original inspiration if applicable]
```

### Why Attribution Matters

- Recognizes valuable contributions to team productivity
- Encourages more sharing (people want credit for good work)
- Creates accountability (if a prompt causes issues, we know who to ask)
- Builds a culture of generosity and collaboration

✅ **Good attribution:** "Prompt by @sarah, improved by @mike — generates 4x faster than manual writing"
❌ **Bad attribution:** Copying someone's prompt into your own collection without credit

---

## Prompt Versioning

Prompts need updates as AI tools and models evolve.

### Version Strategy

```markdown
## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 3.0 | 2025-01-15 | @sarah | Updated for GPT-4o model capabilities |
| 2.1 | 2024-11-20 | @mike | Added GraphQL support |
| 2.0 | 2024-09-01 | @sarah | Restructured for Copilot Chat |
| 1.0 | 2024-06-15 | @sarah | Initial version for Copilot inline |
```

### When to Version

| Trigger | Version Bump | Example |
|---------|-------------|---------|
| New AI model release | Major | Prompt rewritten for new model capabilities |
| New use case added | Minor | GraphQL support added to REST-only prompt |
| Wording tweaks | Patch | Clarified ambiguous instruction |
| Tool update | Major or minor | Copilot Chat API changes |

### Deprecation Process

```markdown
> ⚠️ **DEPRECATED** — This prompt was designed for [old tool/model].
> Use [new-prompt.prompt.md](./new-prompt.prompt.md) instead.
> This file will be removed on YYYY-MM-DD.
```

✅ **Good versioning:** Clear changelog, deprecation notices, migration guidance
❌ **Bad versioning:** Overwriting prompts in place with no history of what changed

---

## Prompt Contribution Template

Use this template when contributing a new prompt to the shared library:

```markdown
---
title: [Descriptive Name]
category: [testing | code-review | documentation | refactoring | debugging | generation]
author: "@username"
created: YYYY-MM-DD
last_tested: YYYY-MM-DD
tools: [copilot-chat, copilot-cli, copilot-inline]
effectiveness: X/5
tags: [tag1, tag2, tag3]
---

# [Prompt Title]

## Purpose
[One paragraph: What does this prompt help you do? When should you use it?]

## The Prompt

> [Exact prompt text — copy-pasteable]

## Context Requirements
[What files, information, or setup does the AI need to produce good results?]

- File 1: [Why it's needed]
- File 2: [Why it's needed]

## Example Usage

### Input
[What the developer provides]

### Output
[Representative example of what the AI produces — sanitized]

## Customization Guide
[How to adapt this prompt for different situations]

| Scenario | Modification |
|----------|-------------|
| [Scenario 1] | [What to change] |
| [Scenario 2] | [What to change] |

## Known Limitations
- [Limitation 1]
- [Limitation 2]

## Testing Results

| Test Scenario | Result | Notes |
|--------------|--------|-------|
| [Scenario 1] | ✅ Pass | [Details] |
| [Scenario 2] | ✅ Pass | [Details] |
| [Scenario 3] | ⚠️ Partial | [What was missing] |

## Related Prompts
- [Related prompt 1](./related-prompt-1.prompt.md)
- [Related prompt 2](./related-prompt-2.prompt.md)
```

---

## Prompt Review Rubric

Use this rubric when reviewing prompt contributions:

### Review Checklist

| Criterion | Weight | Pass | Fail |
|-----------|--------|------|------|
| **Clarity** | High | Prompt is unambiguous and specific | Vague, could be interpreted multiple ways |
| **Tested** | High | Tested on 3+ scenarios with evidence | Tested once or untested |
| **Documented** | Medium | Template fully completed with examples | Missing sections or placeholder text |
| **Scoped** | Medium | Does one thing well | Tries to do too many things |
| **Portable** | Low | Works across similar projects | Only works in one specific repo |
| **Maintained** | Low | Author committed to updates | No maintenance plan |

### Review Decision

- **Approve:** Meets all High criteria and most Medium criteria
- **Request changes:** Fails any High criterion or multiple Medium criteria
- **Reject:** Untested, undocumented, or duplicates an existing prompt

### Review Comment Templates

```markdown
## Prompt Review: [Title]

### Overall Assessment: [Approve | Request Changes | Reject]

### Strengths
- [What's good about this prompt]

### Suggestions
- [ ] [Specific improvement 1]
- [ ] [Specific improvement 2]

### Testing
I tested this prompt in [my environment/project] and found:
- [Result 1]
- [Result 2]
```

✅ **Good review:** Reviewer actually tests the prompt and provides specific, actionable feedback
❌ **Bad review:** "LGTM" without testing the prompt

---

## Organizing Your Prompt Library

### Categorization Taxonomy

```
prompts/
├── by-task/
│   ├── generation/          # Creating new code
│   ├── refactoring/         # Improving existing code
│   ├── testing/             # Writing tests
│   ├── debugging/           # Finding and fixing bugs
│   ├── documentation/       # Writing docs
│   └── review/              # Code review assistance
├── by-language/
│   ├── typescript/
│   ├── python/
│   └── go/
└── by-framework/
    ├── react/
    ├── express/
    └── django/
```

### Prompt Library Index

Maintain a `README.md` that serves as the library's table of contents:

```markdown
# Prompt Library Index

## Most Popular (by usage)
1. [Generate API Test](./testing/api-test.prompt.md) ⭐4.5 — 47 uses
2. [Security Review](./review/security.prompt.md) ⭐4.3 — 38 uses
3. [Component Generator](./generation/react-component.prompt.md) ⭐4.1 — 35 uses

## Recently Added
- [Migration Writer](./generation/db-migration.prompt.md) — @alex, 2025-01-14
- [Error Handler](./generation/error-handler.prompt.md) — @sarah, 2025-01-12

## By Category
### Testing (12 prompts)
- [Unit Test Generator](./testing/unit-test.prompt.md) ⭐4.2
- [Integration Test](./testing/integration-test.prompt.md) ⭐4.5
...
```

✅ **Good organization:** Multiple ways to discover prompts (by popularity, recency, category)
❌ **Bad organization:** Flat list of 100 prompts with no categorization or ranking

---

## Measuring Prompt Sharing Success

| Metric | How to Measure | Healthy Target |
|--------|---------------|----------------|
| Prompts contributed per month | Count PRs to prompt repo | 5+ per team |
| Prompt reuse rate | Usage tracking or survey | > 60% of devs use shared prompts |
| Time from discovery to sharing | Track PR dates | < 1 week |
| Review turnaround | PR metrics | < 2 business days |
| Showcase attendance | Meeting attendance | > 70% of team |
| Cross-team adoption | Forks or copies of prompts | Growing month-over-month |

---

## Next Steps

Continue building your team's knowledge-sharing practices:

- [Internal Documentation](./internal-documentation.md) — Document your team's complete AI workflow practices
- [AI Retrospectives](./retrospectives.md) — Reflect on prompt effectiveness and team learning
- [Community of Practice](./community-of-practice.md) — Scale prompt sharing across the organization

Explore related topics:

- [AI-First Culture](../01-fundamentals/ai-first-culture.md) — Foster the cultural foundation that makes sharing natural
- [Developer Onboarding](../02-onboarding/developer-onboarding.md) — Include prompt sharing in your onboarding program
- [Why Team Workflows Matter](../01-fundamentals/why-team-workflows-matter.md) — Connect prompt sharing to team velocity
- [Role of the Team Lead](../01-fundamentals/role-of-team-lead.md) — Lead by example in sharing effective prompts
- [Maturity Model](../01-fundamentals/maturity-model.md) — Prompt sharing as a maturity indicator
