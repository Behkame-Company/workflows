# Troubleshooting Guide

> Common problems teams encounter with AI-assisted development workflows and their solutions. Each issue includes symptoms, root causes, and step-by-step fixes. Use the decision tree at the end to quickly diagnose your problem.

---

## How to Use This Guide

1. **Find your symptom** in the table of contents below
2. **Follow the diagnosis steps** to identify the root cause
3. **Apply the fix** and verify it resolves the issue
4. **Escalate** if the fix doesn't work — see the escalation path for each issue

---

## Issue 1: Instruction Files Aren't Loading

### Symptoms
- AI suggestions don't follow your team's coding standards
- Copilot ignores patterns defined in your instruction file
- Different team members get wildly different suggestion quality
- AI generates code in the wrong style, framework, or pattern

### Root Causes & Fixes

#### Cause A: Wrong File Path or Name

The instruction file must be in the correct location for each tool.

✅ **Correct paths:**
```
.github/copilot-instructions.md          # GitHub Copilot
.cursorrules                              # Cursor (legacy)
.cursor/rules/                            # Cursor (modern, multiple files)
.clinerules                               # Cline
.claude/                                  # Claude Code
```

❌ **Common mistakes:**
```
github/copilot-instructions.md            # Missing the dot prefix
.github/copilot_instructions.md           # Underscore instead of hyphen
.github/copilot-instructions.txt          # Wrong extension
copilot-instructions.md                   # Wrong directory (root instead of .github/)
```

**Fix:** Verify the exact filename and path for your tool. Check for typos, case sensitivity (on Linux/macOS), and correct directory nesting.

#### Cause B: File Format Issues

✅ **Good format:**
```markdown
## Code Style
- Use TypeScript for all new files
- Use camelCase for variables and functions
- Maximum line length: 100 characters
```

❌ **Bad format:**
```markdown
<!-- This won't work well -->
<instructions>
  <rule>Use TypeScript</rule>
</instructions>
```

**Fix:** Use plain markdown with clear, concise bullet points. AI tools parse markdown natively — don't use XML, JSON, or other structured formats inside the instructions file.

#### Cause C: File Too Long or Too Vague

AI tools have context windows. An instruction file that's 5,000 lines long will be truncated or diluted.

✅ **Good:** 50–200 lines of focused, actionable instructions
❌ **Bad:** 2,000+ lines covering every possible scenario

**Fix:** Keep instructions concise and prioritized. Put the most important rules first. Split into multiple files if your tool supports it (Cursor supports `.cursor/rules/` directory).

#### Cause D: Tool-Specific Configuration Missing

Some tools require additional setup beyond the instruction file.

**GitHub Copilot:**
- Ensure the repository has Copilot enabled in organization settings
- Check that the user has accepted the Copilot Terms of Service
- Verify instruction file is on the default branch (some features require this)

**Cursor:**
- Check Settings → Rules → "Include .cursorrules" is enabled
- For project rules, verify `.cursor/rules/` directory structure

**Cline:**
- Verify `.clinerules` is in the workspace root
- Check Cline settings for custom instructions override

**Fix:** Check tool-specific documentation for configuration requirements beyond the instruction file.

### Verification Steps
1. Open the instruction file and confirm it's valid markdown
2. Restart your editor / reload the AI tool extension
3. Ask the AI a question directly referencing an instruction (e.g., "What naming convention should I use?")
4. If the AI responds with your convention, instructions are loading correctly

---

## Issue 2: AI Output Quality Is Inconsistent

### Symptoms
- Some developers get great suggestions while others get poor ones
- Quality varies by task type (great for CRUD, terrible for algorithms)
- The same prompt produces very different results on different days
- AI generates code that "sort of works" but has subtle bugs

### Root Causes & Fixes

#### Cause A: Vague or Missing Context in Prompts

✅ **Good prompt:**
```
Create a REST endpoint in Express.js that:
- Accepts POST /api/users with { name, email, role }
- Validates email format and role enum (admin, user, viewer)
- Checks for duplicate email in PostgreSQL users table
- Returns 201 with the created user (excluding password hash)
- Returns 400 with specific validation error messages
- Uses our existing errorHandler middleware
```

❌ **Bad prompt:**
```
Create a user endpoint
```

**Fix:** Train team members on prompt engineering. Specific prompts produce specific results. Include input/output examples, constraints, and references to existing code patterns.

#### Cause B: Missing Codebase Context

AI works best when it can see related files. If developers work in isolated files, the AI lacks context.

✅ **Good practice:** Open related files in your editor before prompting
❌ **Bad practice:** Working in a single file with all others closed

**Fix:** Before complex prompts, ensure relevant files are open or referenced:
- Open the interface/type definition file
- Open a similar existing implementation
- Open the test file for the module
- Reference file paths in your prompt: "Follow the pattern in `src/controllers/orders.ts`"

#### Cause C: Model Limitations

Different AI models have different strengths:

| Task Type            | Recommended Approach                              |
|----------------------|---------------------------------------------------|
| Boilerplate/CRUD     | Copilot inline completion (fast, good enough)     |
| Complex logic        | Chat with Claude or GPT-4 (more reasoning)        |
| Refactoring          | Cline/Cursor agent mode (sees full context)        |
| Debugging            | Chat with error messages + stack traces pasted in  |
| Architecture         | Human-driven; use AI for research, not decisions   |

**Fix:** Match the tool to the task. Don't expect inline completion to handle complex architectural decisions.

#### Cause D: Stale Instruction Files

If your codebase has evolved but your instruction file hasn't, the AI will suggest outdated patterns.

✅ **Good:** Instructions updated when frameworks, libraries, or patterns change
❌ **Bad:** Instruction file written 6 months ago and never touched

**Fix:** Add "Review instruction files" to your sprint checklist. Assign the AI Champion to keep them current.

### Verification Steps
1. Compare prompts from developers getting good vs. poor results
2. Check if instruction files are up-to-date with current codebase patterns
3. Verify developers are using the right tool for the right task
4. Test a standardized prompt across team members — if results vary, it's a tool/config issue

---

## Issue 3: Team Members Resist AI Tools

### Symptoms
- Developers avoid using AI tools despite being available
- Complaints about "AI writing bad code" or "it's slower than just typing"
- Some team members feel threatened by AI tools
- Low adoption metrics despite management push

### Root Causes & Fixes

#### Cause A: Fear and Uncertainty

Some developers worry AI will replace them or devalue their skills.

✅ **Effective response:**
- "AI handles the boring parts so you can focus on the interesting problems"
- "Your expertise is what makes AI output actually useful"
- Share concrete examples of how AI amplifies (not replaces) developer skills

❌ **Ineffective response:**
- "Everyone must use AI tools immediately"
- "AI will make us 10x productive"
- Mandating usage without addressing concerns

**Fix:** Hold an open team discussion about concerns. Acknowledge fears honestly. Frame AI as a power tool, not a replacement. Lead by example — have respected senior developers share their workflows.

#### Cause B: Poor First Experience

If a developer's first AI interaction produces garbage, they'll write off the tool.

✅ **Good onboarding:** Guided pair programming with an experienced AI user
❌ **Bad onboarding:** "Here's a license, figure it out"

**Fix:**
1. Use the Developer Onboarding Checklist (see [Templates](./templates.md#template-3-developer-onboarding-checklist))
2. Pair new users with experienced ones for the first week
3. Start with tasks where AI excels (tests, boilerplate, documentation)
4. Gradually expand to more complex tasks as confidence builds

#### Cause C: Workflow Disruption

AI tools that interrupt flow state or slow down existing workflows will be rejected.

✅ **Good integration:** AI fits naturally into existing workflow
❌ **Bad integration:** Forced workflow changes to accommodate AI

**Fix:** Let developers customize their AI interaction style:
- Some prefer inline completions only (minimal disruption)
- Some prefer chat-based interaction (more control)
- Some prefer agent mode (maximum automation)
- Don't mandate a single interaction style

#### Cause D: Legitimate Concerns

Sometimes resistance is based on valid issues — respect that.

**Valid concerns and responses:**
| Concern                                  | Response                                      |
|------------------------------------------|-----------------------------------------------|
| "AI code has security issues"            | Implement quality gates (see Issue 5)          |
| "AI doesn't understand our domain"       | Improve instruction files and context          |
| "I'm faster without AI for this task"    | That's fine — AI isn't always the right tool   |
| "AI-generated code is hard to maintain"  | Fair point — set quality standards for AI code |

**Fix:** Create a safe channel for feedback. Act on legitimate concerns. Don't dismiss valid criticism as "resistance to change."

### Verification Steps
1. Survey the team anonymously about AI tool concerns
2. Check onboarding quality — are new users getting proper guidance?
3. Review whether AI tools are being mandated vs. encouraged
4. Identify workflow friction points and address them

---

## Issue 4: PRs Are Too Large

### Symptoms
- AI-assisted PRs contain hundreds or thousands of changed lines
- Reviewers skip thorough review because PRs are overwhelming
- PR review times increase dramatically
- More bugs slip through review

### Root Causes & Fixes

#### Cause A: AI Makes It Too Easy to Generate Code

When code generation is effortless, developers stop thinking about PR scope.

✅ **Good PR:** 50–200 lines, single concern, reviewable in 15 minutes
❌ **Bad PR:** 2,000 lines, multiple features, takes hours to review

**Fix:** Set explicit PR size limits:
```yaml
# Example: GitHub Action to flag large PRs
name: PR Size Check
on: [pull_request]
jobs:
  size-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Check PR size
        run: |
          LINES=$(git diff --stat origin/main...HEAD | tail -1 | awk '{print $4}')
          if [ "$LINES" -gt 400 ]; then
            echo "::warning::PR has $LINES changed lines. Consider splitting."
          fi
          if [ "$LINES" -gt 800 ]; then
            echo "::error::PR has $LINES changed lines. Must be split."
            exit 1
          fi
```

#### Cause B: Not Breaking Down Tasks

✅ **Good workflow:**
1. Plan the feature as 3–5 small, sequential PRs
2. Each PR is independently reviewable and mergeable
3. Use feature flags if needed for incomplete features

❌ **Bad workflow:**
1. Ask AI to "build the entire feature"
2. Submit everything in one massive PR

**Fix:** Train developers to decompose work BEFORE engaging AI:
- PR 1: Data model and migrations
- PR 2: API endpoints (CRUD)
- PR 3: Business logic and validation
- PR 4: Tests
- PR 5: UI components

#### Cause C: Including Generated Boilerplate

AI often generates more code than necessary — tests for obvious cases, excessive error handling, unused utility functions.

**Fix:** Review AI output critically before committing:
- Remove unnecessary generated code
- Consolidate duplicated patterns
- Ensure generated tests actually test meaningful behavior

### Verification Steps
1. Check average PR size before and after AI adoption
2. Review whether large PRs are the norm or exceptions
3. Survey reviewers about PR quality and reviewability
4. Implement automated PR size checks

---

## Issue 5: AI Introduces Security Vulnerabilities

### Symptoms
- SAST scanners flag AI-generated code
- Security review finds injection vulnerabilities, hardcoded secrets, or insecure defaults
- AI suggests deprecated or vulnerable library versions
- Generated authentication/authorization code has logic flaws

### Root Causes & Fixes

#### Cause A: AI Training Data Includes Insecure Patterns

AI models learn from public code, which includes many insecure examples.

**Common AI security mistakes:**
| Vulnerability               | Example                                                  |
|-----------------------------|----------------------------------------------------------|
| SQL Injection               | String concatenation instead of parameterized queries     |
| XSS                         | Rendering user input without sanitization                 |
| Hardcoded secrets           | `const API_KEY = "sk-abc123..."` in source code          |
| Insecure defaults           | `cors({ origin: '*' })` or `verify: false`               |
| Outdated dependencies       | Suggesting libraries with known CVEs                      |
| Path traversal              | Using user input in file paths without validation         |

**Fix:** Implement automated security scanning as a quality gate:

```yaml
# Security scanning in CI
- name: CodeQL Analysis
  uses: github/codeql-action/analyze@v3
  with:
    languages: javascript, python

- name: Dependency Check
  uses: dependency-check/Dependency-Check_Action@main

- name: Secret Scanning
  uses: gitleaks/gitleaks-action@v2
```

#### Cause B: Developers Trust AI Output Too Much

✅ **Good mindset:** "AI-generated code needs MORE security scrutiny, not less"
❌ **Bad mindset:** "AI wouldn't generate insecure code"

**Fix:** Add security-focused review checklist for AI-assisted PRs:
1. Check all database queries for parameterization
2. Verify input validation on all user-facing endpoints
3. Confirm no secrets in code (use environment variables)
4. Review authentication and authorization logic manually
5. Check dependency versions against known vulnerability databases

#### Cause C: Instruction Files Don't Cover Security

If your instruction file doesn't mention security practices, AI won't prioritize them.

✅ **Add to your instruction file:**
```markdown
## Security Rules
- ALWAYS use parameterized queries (never string concatenation for SQL)
- NEVER hardcode secrets, tokens, or API keys
- ALWAYS validate and sanitize user input
- ALWAYS use HTTPS for external API calls
- Use the principle of least privilege for all data access
- Follow OWASP Top 10 guidelines
```

❌ **Missing from instruction file:** Any mention of security

**Fix:** Add explicit security rules to your instruction file and update them regularly.

### Verification Steps
1. Run SAST scanner on recent AI-assisted PRs
2. Review instruction files for security coverage
3. Check if security training is part of AI onboarding
4. Audit recent AI-generated code for common vulnerability patterns

---

## Issue 6: Can't Measure AI Impact

### Symptoms
- Leadership asks "Is AI worth the investment?" and you can't answer
- No data to compare before/after AI adoption
- Metrics feel arbitrary or gaming-prone
- Different teams measure different things (or nothing at all)

### Root Causes & Fixes

#### Cause A: No Baseline Measurement

You can't show improvement without knowing where you started.

✅ **Good:** Captured baseline metrics for 2-4 weeks before AI rollout
❌ **Bad:** Started measuring only after AI adoption was well underway

**Fix:** If you missed the baseline:
1. Use historical data from your project management tool (velocity, cycle time)
2. Compare teams with AI tools vs. teams without (if applicable)
3. Use "before/after" for individual developers who adopted at different times
4. Start measuring NOW — next quarter you'll have a comparison point

#### Cause B: Measuring the Wrong Things

| ❌ Bad Metrics                    | ✅ Good Metrics                        |
|----------------------------------|----------------------------------------|
| Lines of code written            | Features delivered per sprint           |
| Number of AI prompts used        | Time from ticket to PR                  |
| Copilot acceptance rate          | Bug escape rate (bugs found post-merge) |
| "AI usage hours"                 | Developer satisfaction score            |

**Fix:** Focus on outcome metrics, not activity metrics. See the [ROI Report Template](./templates.md#template-7-roi-report-template) for recommended metrics.

#### Cause C: No Measurement Infrastructure

If collecting metrics requires manual effort, it won't happen consistently.

**Fix:** Automate what you can:
- PR metrics: Extract from GitHub API (size, review time, merge time)
- Build metrics: Extract from CI/CD pipeline (pass rate, duration)
- Deployment metrics: Extract from deployment tools (frequency, failure rate)
- Survey metrics: Schedule recurring surveys (see [Feedback Survey template](./templates.md#template-9-feedback-survey))

### Verification Steps
1. List what metrics you currently track
2. Identify gaps in your measurement framework
3. Check if you have baseline data from before AI adoption
4. Set up automated metric collection where possible

---

## Issue 7: Different Teams Do Different Things

### Symptoms
- No consistency in AI tool usage across the organization
- Team A uses Copilot only; Team B uses Cursor + Claude; Team C avoids AI entirely
- Different PR review standards for AI-assisted code
- No shared instruction file patterns or prompt libraries
- Knowledge silos — teams don't share AI learnings

### Root Causes & Fixes

#### Cause A: No Central Governance

✅ **Good:** Organization-wide AI usage policy with team-level flexibility
❌ **Bad:** Every team invents their own approach from scratch

**Fix:** Implement a governance model with two layers:
1. **Organization level:** Approved tools, security requirements, data classification rules
2. **Team level:** Specific instructions, prompt libraries, workflow customizations

#### Cause B: No Cross-Team Communication

✅ **Good:** Monthly AI Champions meeting, shared prompt library, cross-team demos
❌ **Bad:** Teams operate in isolation with no knowledge sharing

**Fix:**
1. Establish an AI Champions community of practice
2. Create a shared prompt library accessible to all teams
3. Hold monthly "AI Show and Tell" sessions
4. Maintain organization-wide documentation (like this guide!)

#### Cause C: Different Team Maturities

Some teams may be advanced while others are just starting. That's OK.

**Fix:** Use the [Maturity Assessment](./templates.md#template-10-maturity-assessment) to:
1. Understand where each team is on the maturity curve
2. Set team-specific goals (don't force Level 1 teams to Level 4 practices)
3. Pair advanced teams with beginners for knowledge transfer
4. Celebrate progress at every level

### Verification Steps
1. Survey teams on their current AI practices
2. Identify common patterns and divergences
3. Determine what should be standardized vs. team-specific
4. Create a governance framework that balances consistency with flexibility

---

## Issue 8: AI Tools Are Too Expensive

### Symptoms
- License costs seem high relative to perceived benefit
- Leadership questions the ROI of AI tools
- Budget constraints limit rollout to a subset of developers
- Free alternatives seem "good enough"

### Root Causes & Fixes

#### Cause A: Not Measuring ROI

✅ **Good:** "We spend $50/dev/month and save 8 hours/dev/month = $400 value at $50/hour"
❌ **Bad:** "We spend $50/dev/month and... it seems helpful?"

**Fix:** Use the [ROI Report Template](./templates.md#template-7-roi-report-template) to quantify value:
- Track developer hours saved (even rough estimates)
- Calculate cost per hour saved
- Include quality improvements (fewer bugs = less rework)
- Factor in developer satisfaction and retention

#### Cause B: Over-Licensing

Not everyone needs every tool at the highest tier.

✅ **Right-sized licensing:**
| Role                | Tools Needed                              |
|---------------------|-------------------------------------------|
| Full-time developer | Copilot Business + 1 chat tool            |
| Part-time developer | Copilot Business                          |
| Tech lead           | Copilot Business + Claude/Cursor          |
| QA engineer         | Copilot for test writing                  |
| DevOps engineer     | Copilot for IaC and scripts               |

❌ **Over-licensing:** Giving everyone Copilot + Claude + Cursor + Cline

**Fix:** Audit actual usage. Remove licenses for inactive users. Tier your tool access by role and need.

#### Cause C: Not Using Tools Effectively

The tools are only expensive if they don't deliver value. Often the issue is underutilization, not overpricing.

**Fix:**
1. Invest in training (the cost of training is tiny compared to license costs)
2. Assign an AI Champion to drive adoption (see [Template 6](./templates.md#template-6-ai-champion-job-description))
3. Share success stories and techniques across the team
4. Set utilization targets (e.g., "80% of developers using AI tools weekly")

### Verification Steps
1. Calculate your current AI tool spend per developer
2. Estimate hours saved per developer per month
3. Calculate ROI: (hours saved × hourly rate) / tool cost
4. If ROI < 1, focus on training and adoption; if ROI > 3, consider expanding

---

## Decision Tree: Diagnosing Your Problem

Use this text-based flowchart to quickly identify which section to read:

```
START: What's the main symptom?
│
├─► "AI suggestions are bad or inconsistent"
│   ├─► Are instruction files loading?
│   │   ├─► No  → See Issue 1 (Instruction Files Aren't Loading)
│   │   └─► Yes → See Issue 2 (Output Quality Is Inconsistent)
│   │
│   └─► Is it a security issue specifically?
│       └─► Yes → See Issue 5 (Security Vulnerabilities)
│
├─► "Team isn't using the tools"
│   ├─► Is it a cost/access problem?
│   │   └─► Yes → See Issue 8 (Too Expensive)
│   └─► Is it a people/culture problem?
│       └─► Yes → See Issue 3 (Team Members Resist AI Tools)
│
├─► "Process is breaking down"
│   ├─► PRs too large or low quality?
│   │   └─► Yes → See Issue 4 (PRs Are Too Large)
│   ├─► Teams doing different things?
│   │   └─► Yes → See Issue 7 (Different Teams Do Different Things)
│   └─► Can't measure or justify AI adoption?
│       └─► Yes → See Issue 6 (Can't Measure Impact)
│
└─► "Not sure what's wrong"
    └─► Run the Maturity Assessment (templates.md, Template 10)
        and compare scores to identify the weakest area
```

---

## Escalation Paths

When self-service troubleshooting isn't enough:

| Level   | Who                    | When to Escalate                                     |
|---------|------------------------|------------------------------------------------------|
| Level 1 | Team AI Champion       | First stop for any AI tool question or issue          |
| Level 2 | Engineering Manager    | Policy decisions, budget requests, team conflicts     |
| Level 3 | Platform/DevEx Team    | Tool configuration, CI/CD integration, infrastructure |
| Level 4 | Security Team          | Data classification questions, vulnerability reports  |
| Level 5 | Vendor Support         | Tool bugs, outages, licensing issues                  |

---

## Quick Fixes Checklist

Before diving into a full diagnosis, try these quick fixes:

- [ ] Restart your editor / reload extensions
- [ ] Check that your AI tool subscription is active
- [ ] Verify you're on the latest version of the AI tool extension
- [ ] Check the instruction file exists and has no syntax errors
- [ ] Clear AI tool cache (tool-specific; check docs)
- [ ] Check tool status pages for outages
- [ ] Try the same prompt in the AI tool's web interface to isolate IDE issues
- [ ] Ask a teammate to try the same prompt (isolates account/config issues)

---

## Next Steps

- **[Templates Collection](./templates.md)** — Grab templates for the solutions mentioned in this guide
- **[FAQ](./faq.md)** — Quick answers that may solve your issue faster than troubleshooting
- **[Resources & Further Reading](./resources.md)** — Deep-dive documentation for complex issues
- **[Developer Onboarding](../02-onboarding/developer-onboarding.md)** — Fix onboarding issues at the source
- **[AI-First Culture](../01-fundamentals/ai-first-culture.md)** — Address cultural resistance with the right framing
- **[Role of Team Lead](../01-fundamentals/role-of-team-lead.md)** — Leadership strategies for driving adoption
- **[Cowboy AI Development](../10-anti-patterns/cowboy-ai-development.md)** — Recognize anti-patterns that cause these issues
- **[Gradual Adoption](../09-best-practices/gradual-adoption.md)** — Avoid problems by adopting AI tools incrementally
