# Internal Documentation for AI Workflows

> A comprehensive guide to documenting your team's AI-assisted development practices — from tool setup to decision rationale — so knowledge scales beyond any single person.

---

## Why Document AI Practices?

AI-assisted development introduces new tools, conventions, and mental models that evolve rapidly. Without intentional documentation, teams suffer from:

- **Knowledge silos** — only one person knows how to configure Copilot for the monorepo
- **Repeated mistakes** — the same prompt anti-patterns keep resurfacing
- **Slow onboarding** — new hires spend weeks discovering tribal knowledge
- **Inconsistent practices** — each developer uses AI differently with varying quality

Good internal documentation transforms individual discoveries into team-wide capabilities.

### The Documentation Paradox

Teams often resist documentation because "things change too fast." But AI workflows change fast *precisely because* they aren't documented — without a shared baseline, every developer reinvents the wheel.

✅ **Good approach:** Document the *current* best practice with a "last reviewed" date and an owner
❌ **Bad approach:** Avoid documenting anything because "it'll change next month"

---

## What to Document

### 1. Tool Setup Guides

Step-by-step instructions for getting AI tools working in your environment.

#### What to Include

| Element | Description | Example |
|---------|-------------|---------|
| Prerequisites | Required software, accounts, permissions | "Node.js 18+, GitHub Copilot license" |
| Installation steps | Exact commands or UI steps | `code --install-extension GitHub.copilot` |
| Configuration | Settings files, environment variables | `.vscode/settings.json` snippets |
| Verification | How to confirm it's working | "Open a .ts file, type a comment, confirm suggestions appear" |
| Troubleshooting | Common setup failures | "If behind a proxy, set HTTP_PROXY in..." |

#### Example: Tool Setup Document

```markdown
# GitHub Copilot Setup Guide

**Last updated:** 2025-01-15
**Owner:** @DevToolsTeam

## Prerequisites
- VS Code 1.85+
- Active GitHub Copilot Business license (request via IT portal)
- Node.js 18+ (for Copilot CLI)

## Installation

### VS Code Extension
1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X)
3. Search "GitHub Copilot"
4. Install both "GitHub Copilot" and "GitHub Copilot Chat"
5. Sign in with your GitHub account when prompted

### Copilot CLI
```bash
npm install -g @githubnext/github-copilot-cli
github-copilot-cli auth
```

## Configuration
Add to your `.vscode/settings.json`:
```json
{
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": true
  }
}
```

## Verify Installation
1. Open any `.js` or `.ts` file
2. Type `// function to calculate fibonacci`
3. You should see a ghost text suggestion within 1-2 seconds
4. Press Tab to accept

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No suggestions appearing | Check Copilot icon in status bar — click to enable |
| "Not authorized" error | Verify license at github.com/settings/copilot |
| Slow suggestions | Check network; Copilot needs internet access |
```

✅ **Good setup guide:** Includes exact versions, verification steps, and common failure modes
❌ **Bad setup guide:** "Install Copilot and configure it for your needs"

---

### 2. Team Conventions

Document how your team has agreed to use AI tools.

#### Convention Categories

**Instruction Files**
- Where custom instruction files live (e.g., `.github/copilot-instructions.md`)
- What conventions are encoded in them
- Who can modify them and the review process

**Code Review Process**
- How AI-generated code should be marked (if at all)
- Additional review scrutiny for AI-assisted changes
- Which AI review tools are approved

**Commit Practices**
- Co-authored-by trailer conventions
- Commit message standards for AI-assisted work
- Branch naming conventions

#### Example: Team Convention Document

```markdown
# AI Development Conventions

**Last updated:** 2025-01-15
**Owner:** @TechLead

## Instruction Files
- Location: `.github/copilot-instructions.md` in each repository
- Review: Changes require approval from 2 team members
- Scope: Project-specific coding standards and architecture decisions

## Code Review Standards
- All AI-generated code receives the same review as human-written code
- Reviewers should verify: logic correctness, test coverage, security
- Flag patterns that look like AI hallucinations (confident but wrong)

## Commit Conventions
- Include `Co-authored-by: Copilot` trailer when AI substantially contributed
- Commit messages must be human-written (not AI-generated boilerplate)
- Each commit should represent a logical unit of work, not an AI session
```

✅ **Good convention:** "AI-generated code must have tests — no exceptions. The author is responsible for test quality."
❌ **Bad convention:** "Use AI responsibly" (too vague to be actionable)

---

### 3. Prompt Library

A curated collection of tested prompts that work well for your team's codebase and tech stack.

#### Prompt Library Structure

Each prompt entry should include:

```markdown
## Prompt: [Descriptive Name]

**Category:** [Code Generation | Refactoring | Testing | Debugging | Documentation]
**Tool:** [Copilot Chat | Copilot CLI | Custom Skill]
**Added by:** @username
**Last tested:** YYYY-MM-DD
**Effectiveness:** ⭐⭐⭐⭐⭐ (5/5)

### The Prompt
> [Exact prompt text here]

### When to Use
[Describe the situation where this prompt excels]

### Example Input
[What the developer provides as context]

### Example Output
[What the AI typically produces — sanitized/generalized]

### Tips
- [Any nuances or modifications that help]

### Known Limitations
- [Where this prompt fails or produces poor results]
```

#### Example Prompt Library Entries

```markdown
## Prompt: Generate API Integration Test

**Category:** Testing
**Tool:** Copilot Chat
**Added by:** @sarah
**Last tested:** 2025-01-10
**Effectiveness:** ⭐⭐⭐⭐ (4/5)

### The Prompt
> Write an integration test for the [endpoint] API endpoint.
> Test happy path, validation errors, authentication failures,
> and rate limiting. Use our test helpers from `src/test/helpers.ts`.
> Follow the Arrange-Act-Assert pattern.

### When to Use
When adding a new API endpoint or backfilling test coverage.

### Example Input
Provide the route handler file and any relevant DTOs/schemas.

### Tips
- Include the actual test helper file in context for better results
- Specify the test framework explicitly if not obvious from context

### Known Limitations
- Struggles with complex database setup/teardown
- May not match your exact assertion style — review carefully
```

✅ **Good prompt library:** Organized by category, tested with dates, includes known limitations
❌ **Bad prompt library:** Random list of prompts with no context or testing information

---

### 4. Troubleshooting Guide

A living document of problems encountered and solutions discovered.

#### Troubleshooting Entry Format

```markdown
## Problem: [Brief Description]

**Symptoms:** [What the developer observes]
**Root Cause:** [Why it happens]
**Solution:** [Step-by-step fix]
**Prevention:** [How to avoid it in the future]
**Reported by:** @username
**Date:** YYYY-MM-DD
```

#### Common Troubleshooting Categories

| Category | Example Issues |
|----------|---------------|
| Tool configuration | Copilot not suggesting in certain file types |
| Context problems | AI not aware of project conventions |
| Quality issues | Generated code doesn't match team style |
| Performance | Slow suggestions, timeouts |
| Security | AI suggesting hardcoded credentials |
| Integration | AI tools conflicting with linters/formatters |

#### Example Troubleshooting Entries

```markdown
## Problem: Copilot Ignores Custom Instructions

**Symptoms:** AI suggestions don't follow patterns defined in
`.github/copilot-instructions.md`.

**Root Cause:** The instruction file has grown beyond the effective
context window. Only the first ~500 lines are reliably used.

**Solution:**
1. Audit the instruction file for redundancy
2. Prioritize the most important conventions at the top
3. Move detailed examples to separate `.prompt.md` files
4. Keep the main instruction file under 300 lines

**Prevention:** Review instruction file size during quarterly audits.

**Reported by:** @mike
**Date:** 2025-01-08
```

✅ **Good troubleshooting entry:** Specific symptoms, verified root cause, actionable solution
❌ **Bad troubleshooting entry:** "Copilot wasn't working, then it started working again"

---

### 5. Decision Log

Record *why* the team made specific AI-related decisions.

#### Decision Log Format

```markdown
## Decision: [Title]

**Date:** YYYY-MM-DD
**Status:** [Accepted | Superseded | Deprecated]
**Participants:** @person1, @person2

### Context
[What situation prompted this decision?]

### Options Considered
1. **Option A** — [Brief description] — Pros: ... Cons: ...
2. **Option B** — [Brief description] — Pros: ... Cons: ...

### Decision
[What we decided and why]

### Consequences
[What this means for the team going forward]
```

#### Example Decision Log Entry

```markdown
## Decision: Require Human-Written Commit Messages

**Date:** 2025-01-05
**Status:** Accepted
**Participants:** @techlead, @senior1, @senior2

### Context
Several developers started using AI to generate commit messages.
While convenient, the messages were often generic ("fix bug",
"update code") and didn't capture the *why* behind changes.

### Options Considered
1. **Allow AI commit messages** — Fast, low friction — but poor history
2. **Ban AI commit messages** — Better history — but slower workflow
3. **AI-assisted with human editing** — Good balance — requires discipline

### Decision
Option 3: Developers may use AI to draft commit messages but must
edit them to include the reasoning behind the change. The first
line must be human-authored.

### Consequences
- Commit messages should improve in quality
- Slight additional time per commit (~30 seconds)
- Need to add this to onboarding documentation
```

✅ **Good decision log:** Captures context, alternatives, and rationale for future reference
❌ **Bad decision log:** Only records the decision without explaining why

---

### 6. FAQ

Answers to questions that come up repeatedly.

#### Example FAQ Entries

```markdown
## Frequently Asked Questions

### General

**Q: Can I use AI to write production code?**
A: Yes, with the same quality standards as human-written code.
All AI-assisted code must pass code review, have tests, and
meet our security requirements.

**Q: Do I need to disclose that I used AI?**
A: Add the `Co-authored-by: Copilot` trailer to commits where
AI substantially contributed. You don't need to flag every
single-line autocomplete.

**Q: What data does Copilot send to the cloud?**
A: Code context from your current file and open tabs. With
Copilot Business, your code is not stored or used for training.
See our security policy for details.

### Practical

**Q: Copilot keeps suggesting code in the wrong style. What do I do?**
A: Check that `.github/copilot-instructions.md` is up to date
with our coding conventions. Include a few examples of our
preferred patterns.

**Q: Can I use AI for security-sensitive code?**
A: Yes, but with additional review requirements. Security-critical
code (auth, crypto, input validation) requires review from
a security champion regardless of how it was written.

**Q: How do I report an AI-related incident?**
A: Use the #ai-incidents Slack channel or email security@company.com.
Include: what happened, what tool was used, and the impact.
```

✅ **Good FAQ:** Answers the actual question directly with actionable guidance
❌ **Bad FAQ:** "Q: How do I use AI? A: Please refer to the documentation."

---

## Where to Store Documentation

### Option 1: Docs Folder in Repository (Recommended)

```
repo/
├── .github/
│   └── copilot-instructions.md    # AI reads this automatically
├── docs/
│   ├── ai/
│   │   ├── setup-guide.md
│   │   ├── conventions.md
│   │   ├── prompt-library.md
│   │   ├── troubleshooting.md
│   │   ├── decisions/
│   │   │   ├── 001-commit-messages.md
│   │   │   └── 002-test-requirements.md
│   │   └── faq.md
│   └── ...
```

**Pros:** Version controlled, reviewed via PR, close to the code
**Cons:** Scattered across repos in multi-repo setups

### Option 2: Internal Wiki or Confluence

Best for cross-team or organization-wide documentation.

**Pros:** Centralized, searchable, accessible to non-engineers
**Cons:** Not version controlled, can become stale

### Option 3: Hybrid Approach (Recommended for Larger Teams)

| Documentation Type | Location | Reason |
|-------------------|----------|--------|
| Tool setup | Wiki | Applies across all repos |
| Team conventions | Repo `docs/` | Specific to each project |
| Prompt library | Shared repo | Reusable across projects |
| Troubleshooting | Wiki | Cross-project issues |
| Decision log | Repo `docs/` | Context-specific decisions |
| FAQ | Wiki | General questions |

✅ **Good approach:** Match the documentation location to its scope and audience
❌ **Bad approach:** Put everything in one place regardless of who needs it

---

## Documentation Maintenance

### Assign Documentation Owners

Every document needs a named owner responsible for keeping it current.

```markdown
## Documentation Ownership Registry

| Document | Owner | Backup | Review Cycle |
|----------|-------|--------|-------------|
| Setup Guide | @devtools-team | @techlead | Quarterly |
| Conventions | @techlead | @senior-dev | Monthly |
| Prompt Library | @ai-champion | Rotating | Biweekly |
| Troubleshooting | @on-call | @devtools-team | As needed |
| Decision Log | @techlead | @product-owner | Per decision |
| FAQ | @ai-champion | @techlead | Monthly |
```

### Quarterly Review Process

```markdown
## Quarterly Documentation Review Checklist

### Preparation (Week Before)
- [ ] Send review reminder to all doc owners
- [ ] Pull usage analytics (page views, search queries)
- [ ] Collect feedback from recent new hires

### Review Meeting (60 minutes)
- [ ] Walk through each document category
- [ ] Identify stale content (>3 months without update)
- [ ] Review open questions from FAQ
- [ ] Prioritize updates for next quarter

### Follow-Up (Week After)
- [ ] Create tickets for identified updates
- [ ] Archive deprecated content
- [ ] Announce changes to team
```

### Update Triggers

Update documentation when any of these occur:

- New AI tool adopted or upgraded
- Team convention changes
- New troubleshooting issue discovered (and resolved)
- New team member completes onboarding (capture their feedback)
- Quarterly review cycle
- Significant AI model update (new capabilities or behavior changes)

✅ **Good maintenance:** "Setup guide updated after Copilot 1.200 release — new auth flow documented"
❌ **Bad maintenance:** Documentation written once during initial setup and never touched again

---

## AI-Assisted Documentation

Use AI itself to help create and maintain documentation.

### Using AI to Draft Documentation

```
Prompt: "Review the code in src/auth/ and generate a developer
guide explaining the authentication flow. Include:
- Architecture overview
- Key files and their responsibilities
- Common modification scenarios
- Testing approach
Use our existing docs/TEMPLATE.md format."
```

### Using AI to Identify Documentation Gaps

```
Prompt: "Compare our docs/ai/setup-guide.md with the actual
configuration files in .vscode/ and .github/. Are there any
settings or tools we use that aren't documented?"
```

### Using AI to Keep Documentation Current

```
Prompt: "Review the git log for changes to .github/copilot-instructions.md
in the last 3 months. Summarize what changed and check if our
docs/ai/conventions.md reflects these changes."
```

✅ **Good AI use:** AI drafts documentation that a human reviews, edits, and approves
❌ **Bad AI use:** AI generates documentation that goes directly to the wiki without review

---

## Documentation Templates

### Template: Setup Guide

```markdown
# [Tool Name] Setup Guide

**Last updated:** YYYY-MM-DD
**Owner:** @team-or-person
**Applies to:** [OS/IDE/environment specifics]

## Prerequisites
- [ ] Requirement 1
- [ ] Requirement 2

## Installation
### Step 1: [Description]
[Commands or instructions]

### Step 2: [Description]
[Commands or instructions]

## Configuration
[Settings, environment variables, config files]

## Verification
[How to confirm everything is working]

## Troubleshooting
| Symptom | Cause | Fix |
|---------|-------|-----|
| ... | ... | ... |

## Related Documentation
- [Link to conventions]
- [Link to prompt library]
```

### Template: Convention Document

```markdown
# [Convention Name]

**Last updated:** YYYY-MM-DD
**Owner:** @team-or-person
**Status:** [Active | Draft | Deprecated]

## Summary
[One-paragraph description of this convention]

## Rules
1. **[Rule]** — [Explanation]
2. **[Rule]** — [Explanation]

## Examples

### ✅ Correct
[Example of following the convention]

### ❌ Incorrect
[Example of violating the convention]

## Exceptions
[When this convention doesn't apply]

## Rationale
[Why this convention exists — link to decision log]
```

### Template: Decision Record

```markdown
# Decision: [Title]

**Date:** YYYY-MM-DD
**Status:** [Proposed | Accepted | Deprecated | Superseded by DR-XXX]
**Deciders:** @person1, @person2

## Context
[What is the issue that we're seeing that is motivating this decision?]

## Options

### Option 1: [Name]
- **Pros:** ...
- **Cons:** ...

### Option 2: [Name]
- **Pros:** ...
- **Cons:** ...

## Decision
[Which option did we choose and why?]

## Consequences
[What becomes easier or harder as a result of this decision?]
```

---

## Documentation Quality Checklist

Use this checklist when creating or reviewing AI workflow documentation:

### Content Quality
- [ ] Answers "why" not just "what" and "how"
- [ ] Includes concrete examples (not just abstract concepts)
- [ ] Has clear ✅/❌ examples where appropriate
- [ ] Written for the target audience (not too basic, not too advanced)
- [ ] Free of jargon or jargon is defined

### Maintenance
- [ ] Has a named owner
- [ ] Has a "last updated" date
- [ ] Has a defined review cycle
- [ ] Includes version/tool version it applies to

### Discoverability
- [ ] Has a clear, descriptive title
- [ ] Is linked from a central index or README
- [ ] Uses consistent naming conventions
- [ ] Tagged or categorized for search

### Accuracy
- [ ] All commands and code snippets have been tested
- [ ] Screenshots are current (if included)
- [ ] Links are not broken
- [ ] Aligns with current team practices (not aspirational)

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Harmful | Better Approach |
|-------------|-----------------|-----------------|
| Write-once documentation | Becomes stale and misleading | Assign owners and review cycles |
| Documentation as gatekeeping | Creates bottlenecks | Make contribution easy with templates |
| Over-documenting | Nobody reads 50-page guides | Focus on what people actually need |
| Under-documenting | Knowledge stays tribal | Document decisions and conventions |
| Aspirational documentation | Describes ideal, not reality | Document current state, note goals separately |
| Copy-paste from vendor docs | Duplicates without adding value | Link to vendor docs, document team-specific additions |

---

## Measuring Documentation Effectiveness

Track these metrics to know if your documentation is working:

| Metric | How to Measure | Target |
|--------|---------------|--------|
| Onboarding time | Days until new hire's first AI-assisted PR | < 3 days |
| Repeat questions | Same question asked in Slack more than twice | 0 (add to FAQ) |
| Doc freshness | % of docs updated within their review cycle | > 80% |
| Usage | Page views or search hits (if wiki) | Trending up |
| Satisfaction | Survey new hires after 30 days | > 4/5 rating |
