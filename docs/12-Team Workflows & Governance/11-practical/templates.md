# Templates Collection

> Ready-to-use templates for team AI governance. Copy, customize, and deploy these templates to accelerate your team's AI adoption journey. Each template includes instructions for customization and is designed to work standalone or as part of a complete governance framework.

---

## How to Use These Templates

1. **Browse** the templates below to find what you need
2. **Copy** the template into your own repository or wiki
3. **Customize** the bracketed placeholders (`[Your Company]`, `[Team Name]`, etc.)
4. **Review** with your team and iterate
5. **Publish** to your team's documentation system

> **Tip:** Start with Template 1 (AI Usage Policy) and Template 3 (Developer Onboarding Checklist) — they provide the foundation for everything else.

---

## Template 1: AI Usage Policy (One-Page)

Use this as a quick, accessible policy that every developer can reference. Pin it in Slack, print it, or add it to your wiki.

### Template

```markdown
# [Your Company] AI-Assisted Development Policy
**Effective Date:** [Date] | **Version:** [1.0] | **Owner:** [Name/Role]

## Purpose
This policy establishes guidelines for using AI coding assistants
(GitHub Copilot, Claude, Cline, Cursor, etc.) at [Your Company].

## Approved Tools
| Tool            | Status   | License Type | Approved Use Cases       |
|-----------------|----------|--------------|--------------------------|
| GitHub Copilot  | Approved | Business     | Code completion, chat    |
| Claude          | Approved | Team         | Code review, generation  |
| Cline           | Approved | Pro          | Agentic workflows        |
| Cursor          | Approved | Business     | AI-first editing         |
| [Other]         | [Status] | [Type]       | [Use Cases]              |

## Core Rules
1. **You own the code.** AI-generated code is YOUR responsibility.
2. **Review everything.** Never merge AI output without human review.
3. **No secrets.** Never paste API keys, passwords, or PII into AI tools.
4. **Disclose AI use.** Tag PRs with AI involvement per our PR template.
5. **Follow standards.** AI code must meet the same quality bar as human code.

## Data Classification
| Classification | Can Share with AI? | Examples                     |
|----------------|--------------------|------------------------------|
| Public         | ✅ Yes             | Open-source code, docs       |
| Internal       | ✅ Yes             | Internal APIs, business logic|
| Confidential   | ⚠️ With caution    | Customer schemas (anonymized)|
| Restricted     | ❌ Never           | PII, secrets, credentials    |

## Escalation Path
Questions? → #ai-tools Slack channel → [AI Champion Name] → [Engineering Manager]

## Acknowledgment
By using AI tools at [Your Company], you agree to follow this policy.

Signed: _____________ Date: _____________
```

### Customization Instructions

- Replace all `[bracketed]` values with your organization's specifics
- Adjust the approved tools table to match your actual tooling
- Modify data classification to align with your existing data governance
- Add or remove core rules based on your regulatory requirements

---

## Template 2: Team Instruction File (copilot-instructions.md Starter)

This file goes in `.github/copilot-instructions.md` at the root of your repository. It configures how GitHub Copilot behaves for your team.

### Template

```markdown
# Project Coding Instructions for AI Assistants

## Project Overview
- **Project:** [Project Name]
- **Language(s):** [TypeScript, Python, Go, etc.]
- **Framework(s):** [React, Django, Gin, etc.]
- **Architecture:** [Monolith, Microservices, Serverless, etc.]

## Code Style
- Follow [style guide link] for all code
- Use [tabs/spaces] with [N] space indentation
- Maximum line length: [80/100/120] characters
- Use [single/double] quotes for strings

## Naming Conventions
- **Files:** [kebab-case/camelCase/PascalCase]
- **Variables:** [camelCase]
- **Functions:** [camelCase]
- **Classes:** [PascalCase]
- **Constants:** [UPPER_SNAKE_CASE]
- **Database tables:** [snake_case]
- **API endpoints:** [kebab-case]

## Architecture Patterns
- Use [repository/service/controller] pattern for [backend/API]
- State management: [Redux/Zustand/Context/Pinia]
- Error handling: [describe your error handling approach]
- Logging: Use [Winston/Pino/structured logging] for all log output

## Testing Requirements
- Unit tests required for all [public methods/exported functions]
- Test framework: [Jest/Pytest/Go test]
- Minimum coverage: [80%/90%]
- Test file location: [adjacent/__tests__/test/ directory]
- Test naming: [describe convention, e.g., "should_[action]_when_[condition]"]

## Security Rules
- NEVER hardcode secrets, tokens, or API keys
- ALWAYS use parameterized queries for database access
- ALWAYS validate and sanitize user input
- Use [authentication library] for auth
- Follow OWASP Top 10 guidelines

## Documentation
- All public APIs must have [JSDoc/docstring/GoDoc] comments
- README required for each [service/package/module]
- Include usage examples in documentation

## Off-Limits
- Do NOT modify files in [/config/production, /migrations, etc.]
- Do NOT change the database schema without migration files
- Do NOT introduce new dependencies without team approval

## Common Patterns
[Include 2-3 small code examples showing your team's preferred patterns
for common tasks like API endpoints, database queries, error handling, etc.]
```

### Customization Instructions

- Start with just the sections relevant to your project
- Add real code examples from your codebase for the "Common Patterns" section
- Update this file as your team's conventions evolve
- Keep it concise — AI assistants work better with focused instructions

✅ **Good:** Specific, actionable instructions with examples
❌ **Bad:** Vague guidelines like "write clean code" or "follow best practices"

---

## Template 3: Developer Onboarding Checklist

Use this checklist when onboarding new team members to your AI-assisted workflow.

### Template

```markdown
# AI Tools Onboarding Checklist
**New Developer:** [Name]
**Onboarding Buddy:** [Name]
**Start Date:** [Date]

## Week 1: Setup & Basics

### Tool Installation
- [ ] GitHub Copilot extension installed and authenticated
- [ ] [Claude/Cline/Cursor] installed and configured
- [ ] Verified license assignment in admin portal
- [ ] Connected to organization's AI tool settings

### Configuration
- [ ] Cloned repository with `.github/copilot-instructions.md`
- [ ] Reviewed team instruction file and understood conventions
- [ ] Configured personal AI tool settings per team standard
- [ ] Installed recommended VS Code extensions: [list]
- [ ] Set up any MCP servers the team uses: [list]

### Policy Review
- [ ] Read and signed the AI Usage Policy
- [ ] Completed data classification training
- [ ] Understands what data CAN and CANNOT be shared with AI
- [ ] Knows the escalation path for AI-related questions

## Week 2: Guided Practice

### Pair Programming Sessions
- [ ] Session 1: AI-assisted code completion basics (1 hour)
- [ ] Session 2: Using AI chat for code understanding (1 hour)
- [ ] Session 3: AI-assisted PR creation and review (1 hour)
- [ ] Session 4: Agentic workflow with [Cline/Cursor] (1 hour)

### Solo Practice
- [ ] Completed a small feature using AI assistance
- [ ] Submitted first AI-assisted PR with proper disclosure
- [ ] Reviewed an AI-assisted PR from a teammate
- [ ] Practiced prompt engineering with team examples

## Week 3: Integration & Feedback

### Advanced Usage
- [ ] Explored team prompt library in [location]
- [ ] Learned when NOT to use AI (complex architecture, novel design)
- [ ] Practiced debugging AI-generated code
- [ ] Understands how to file issues with AI tool behavior

### Feedback
- [ ] Completed the AI tools feedback survey
- [ ] Shared what worked well and what was confusing
- [ ] Added at least one prompt to the team prompt library
- [ ] Knows how to request help in #ai-tools channel

## Sign-Off
- [ ] Buddy confirms developer is comfortable with AI workflow
- [ ] Developer confirms they understand the AI usage policy
- [ ] Manager approves onboarding completion

**Buddy Signature:** _____________ **Date:** _____________
**Developer Signature:** _____________ **Date:** _____________
```

### Customization Instructions

- Adjust the timeline to match your onboarding process (some teams compress to 1 week)
- Replace tool names with your actual approved tools
- Add links to your internal documentation, Slack channels, etc.
- Include specific pair programming exercises from your codebase

---

## Template 4: PR Review Template (AI-Disclosure)

Add this to your repository's `.github/PULL_REQUEST_TEMPLATE.md` or use it as a PR description guide.

### Template

```markdown
## Description
[Describe what this PR does and why]

## AI Assistance Disclosure
<!-- Check all that apply -->
- [ ] No AI tools were used in this PR
- [ ] AI used for code generation (specify tool: _______)
- [ ] AI used for test generation (specify tool: _______)
- [ ] AI used for documentation (specify tool: _______)
- [ ] AI used for code review/refactoring (specify tool: _______)
- [ ] AI used for commit messages (specify tool: _______)

### AI Usage Details
<!-- If AI was used, briefly describe how -->
**Tool(s) used:** [e.g., GitHub Copilot, Claude, Cline]
**Estimated AI contribution:** [Low/Medium/High]
**What AI generated:** [e.g., "Boilerplate CRUD endpoints", "Unit tests"]
**What I modified after generation:** [e.g., "Fixed error handling, added edge cases"]

## Review Focus Areas
<!-- Guide reviewers to areas needing extra attention -->
- [ ] AI-generated code has been manually reviewed by the author
- [ ] Security implications have been considered
- [ ] Edge cases and error handling are adequate
- [ ] Tests cover the AI-generated code paths
- [ ] No hardcoded secrets or sensitive data

## Type of Change
- [ ] Bug fix (non-breaking change)
- [ ] New feature (non-breaking change)
- [ ] Breaking change
- [ ] Documentation update
- [ ] Refactoring (no functional changes)

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing performed
- [ ] Test coverage maintained/improved

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Documentation updated (if applicable)
- [ ] No new warnings introduced
```

### Customization Instructions

- Integrate with your existing PR template (add the AI section, don't replace everything)
- Make AI disclosure required or optional based on your governance level
- Add links to your specific review guidelines
- Consider automating AI detection with tools like GitHub Actions

✅ **Good:** "AI used for boilerplate CRUD — I added custom validation and error handling"
❌ **Bad:** "Used Copilot" (too vague to be useful for reviewers)

---

## Template 5: Sprint Retrospective Guide (AI Section)

Add this section to your existing retrospective template to track AI adoption progress.

### Template

```markdown
# Sprint [N] Retrospective — AI Tools Section
**Date:** [Date] | **Facilitator:** [Name] | **Attendees:** [Names]

## AI Usage This Sprint

### Metrics Snapshot
| Metric                          | This Sprint | Last Sprint | Trend |
|---------------------------------|-------------|-------------|-------|
| PRs with AI assistance          | [N]/[Total] | [N]/[Total] | ↑↓→   |
| Avg time to first PR review     | [hours]     | [hours]     | ↑↓→   |
| AI-related bugs found in review | [N]         | [N]         | ↑↓→   |
| Team members actively using AI  | [N]/[Total] | [N]/[Total] | ↑↓→   |
| Estimated hours saved           | [N]         | [N]         | ↑↓→   |

### What Worked Well with AI? 🎉
<!-- Team members share positive AI experiences this sprint -->
1. [Example: "Copilot saved me hours on the data migration boilerplate"]
2. [Example: "Claude helped me understand the legacy billing module"]
3.

### What Didn't Work Well with AI? 😤
<!-- Capture pain points and frustrations -->
1. [Example: "AI-generated tests had poor edge case coverage"]
2. [Example: "Copilot kept suggesting deprecated API patterns"]
3.

### What Should We Try Next Sprint? 🧪
<!-- Experiments to improve AI usage -->
1. [Example: "Update instruction file with new API patterns"]
2. [Example: "Try Cline for the database migration task"]
3.

### Action Items
| Action                              | Owner     | Due Date   |
|-------------------------------------|-----------|------------|
| [e.g., Update copilot-instructions] | [Name]    | [Date]     |
| [e.g., Share prompt for test gen]   | [Name]    | [Date]     |

### AI Adoption Health Check
Rate 1-5 (1=struggling, 5=thriving):
- Tool proficiency across team: [1-5]
- Quality of AI-generated code: [1-5]
- Team confidence in AI workflow: [1-5]
- Instruction file effectiveness: [1-5]
```

### Customization Instructions

- Add this as a recurring section in your existing retro template
- Adjust metrics to match what you can actually measure
- Keep it lightweight — 10 minutes max in the retrospective
- Track trends over time to show AI adoption maturity

---

## Template 6: AI Champion Job Description

Use this to formally define the AI Champion role within your team.

### Template

```markdown
# AI Champion — Role Description
**Team:** [Team Name] | **Reports to:** [Engineering Manager]
**Time Commitment:** ~[4-8] hours/week (part of existing role)

## Purpose
The AI Champion drives effective AI tool adoption within the team,
serving as the go-to resource for AI-assisted development practices.

## Responsibilities

### Weekly
- Monitor team AI usage and identify improvement opportunities
- Answer questions in #ai-tools channel (or redirect appropriately)
- Review and update team instruction files as needed
- Share one useful AI tip or prompt with the team

### Monthly
- Curate team prompt library (add new prompts, retire stale ones)
- Review AI-related metrics and report trends
- Facilitate AI section of sprint retrospective
- Evaluate new AI tools, features, or updates for team relevance

### Quarterly
- Lead AI maturity assessment (see Template 10)
- Update AI usage policy based on team feedback
- Present AI impact report to leadership (see Template 7)
- Coordinate with AI Champions from other teams

## Skills & Qualities
- **Required:**
  - Active user of at least 2 approved AI tools
  - Strong understanding of team coding standards
  - Good communication and teaching skills
  - Willingness to experiment and share findings
- **Preferred:**
  - Experience with prompt engineering
  - Understanding of AI/ML fundamentals
  - Previous mentoring or coaching experience

## Success Metrics
- Team AI adoption rate increases quarter over quarter
- Reduction in AI-related bugs found in review
- Positive feedback from team AI satisfaction surveys
- Growth of team prompt library

## Support Provided
- Dedicated time allocation for AI Champion duties
- Budget for AI tool experimentation: $[amount]/quarter
- Access to AI Champion community of practice
- Training opportunities for new AI tools and techniques
```

### Customization Instructions

- Adjust time commitment based on team size and AI maturity
- Align responsibilities with your team's specific needs
- Add or remove skills based on your technology stack
- Consider rotating the role every 6 months to spread knowledge

---

## Template 7: ROI Report Template

Use this quarterly template to demonstrate the value of AI tool investment.

### Template

```markdown
# AI Tools ROI Report — [Quarter] [Year]
**Prepared by:** [Name] | **Date:** [Date] | **Team:** [Team Name]

## Executive Summary
[2-3 sentences summarizing the quarter's AI impact.
Example: "AI tools saved an estimated 120 developer-hours this quarter,
representing a 15% productivity improvement. Quality metrics remained
stable with a slight improvement in test coverage."]

## Investment
| Cost Category            | Quarterly Cost |
|--------------------------|----------------|
| GitHub Copilot licenses  | $[amount]      |
| Claude/Cline licenses    | $[amount]      |
| Cursor licenses          | $[amount]      |
| Training & onboarding    | $[amount]      |
| AI Champion time (est.)  | $[amount]      |
| **Total Investment**     | **$[amount]**  |

## Returns

### Productivity Gains
| Metric                          | Before AI  | After AI   | Change   |
|---------------------------------|------------|------------|----------|
| Avg PRs per developer per week  | [N]        | [N]        | +[N]%    |
| Avg time from start to PR       | [hours]    | [hours]    | -[N]%    |
| Lines of test code per feature  | [N]        | [N]        | +[N]%    |
| Boilerplate tasks automated     | [N/A]      | [N]/sprint | New      |

### Quality Metrics
| Metric                          | Before AI  | After AI   | Change   |
|---------------------------------|------------|------------|----------|
| Bugs per release                | [N]        | [N]        | [+/-N]%  |
| Code review turnaround          | [hours]    | [hours]    | -[N]%    |
| Test coverage                   | [N]%       | [N]%       | +[N]%    |
| Security findings per scan      | [N]        | [N]        | [+/-N]%  |

### Developer Satisfaction
| Survey Question (1-5 scale)     | Score   | Previous | Trend |
|---------------------------------|---------|----------|-------|
| "AI tools help me be productive"| [N]     | [N]      | ↑↓→   |
| "I trust AI-generated code"     | [N]     | [N]      | ↑↓→   |
| "AI tools are well-integrated"  | [N]     | [N]      | ↑↓→   |

## ROI Calculation
- **Total Investment:** $[amount]
- **Estimated Hours Saved:** [N] hours × $[rate]/hour = $[amount]
- **Quality Cost Avoidance:** $[amount] (fewer bugs, faster reviews)
- **Net ROI:** [N]% = (Returns - Investment) / Investment × 100

## Recommendations
1. [e.g., "Expand Copilot Business to QA team"]
2. [e.g., "Invest in advanced prompt engineering training"]
3. [e.g., "Evaluate Cursor for frontend team"]

## Next Quarter Goals
1. [Goal with measurable target]
2. [Goal with measurable target]
3. [Goal with measurable target]
```

### Customization Instructions

- Use real numbers — even rough estimates are better than nothing
- Compare against your baseline (measure BEFORE AI adoption)
- Adjust the survey questions to your team's priorities
- Present to leadership within 2 weeks of quarter end

✅ **Good:** "Estimated 120 hours saved based on PR velocity increase of 23%"
❌ **Bad:** "AI makes us faster" (no data, not actionable)

---

## Template 8: Quality Gate Configuration

Use this to define automated quality gates for AI-assisted code.

### Template

```markdown
# Quality Gates for AI-Assisted Code
**Last Updated:** [Date] | **Owner:** [Team/Name]

## Gate 1: Pre-Commit (Local)
**Trigger:** Developer runs `git commit`
**Automated Checks:**
- [ ] Linter passes (zero errors)
- [ ] Formatter applied (Prettier/Black/gofmt)
- [ ] No secrets detected (git-secrets / gitleaks)
- [ ] No TODO/FIXME without issue reference
- [ ] Pre-commit hooks pass

**Configuration:**
\`\`\`yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
\`\`\`

## Gate 2: Pull Request (CI/CD)
**Trigger:** PR opened or updated
**Automated Checks:**
- [ ] All unit tests pass
- [ ] Code coverage ≥ [80]% (no decrease)
- [ ] SAST scan clean (CodeQL / Semgrep / Snyk)
- [ ] Dependency vulnerability scan passes
- [ ] Build succeeds on all target platforms
- [ ] PR template completed (AI disclosure section filled)
- [ ] Branch is up to date with main

**Configuration:**
\`\`\`yaml
# .github/workflows/pr-quality-gate.yml
name: PR Quality Gate
on: [pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test -- --coverage
      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi
      - name: SAST Scan
        uses: github/codeql-action/analyze@v3
      - name: Secret Scan
        uses: gitleaks/gitleaks-action@v2
\`\`\`

## Gate 3: Code Review (Human)
**Trigger:** PR passes automated gates
**Required Reviews:** [1-2] approvals
**Review Focus for AI-Assisted Code:**
- [ ] Logic correctness (AI often gets logic subtly wrong)
- [ ] Edge case handling (AI tends to handle the happy path well)
- [ ] Error handling patterns match team standards
- [ ] No over-engineering (AI sometimes adds unnecessary abstractions)
- [ ] Performance considerations (AI may not optimize for your scale)
- [ ] Security review (especially for auth, input handling, data access)

## Gate 4: Pre-Deploy (Staging)
**Trigger:** Merge to main / release branch
**Automated Checks:**
- [ ] Integration tests pass
- [ ] E2E tests pass
- [ ] Performance benchmarks within tolerance
- [ ] Staging deployment succeeds
- [ ] Smoke tests pass on staging

## Gate 5: Post-Deploy (Production)
**Trigger:** Production deployment
**Monitoring:**
- [ ] Error rate monitoring (alert if >1% increase)
- [ ] Performance monitoring (alert if p99 >20% regression)
- [ ] Feature flag gradual rollout configured
- [ ] Rollback plan documented and tested
```

### Customization Instructions

- Adjust coverage thresholds to your team's standards
- Add or remove gates based on your deployment pipeline
- Configure specific SAST tools for your language stack
- Consider stricter gates for AI-heavy PRs (e.g., require 2 reviewers)

✅ **Good:** Automated gates catch issues before human review
❌ **Bad:** Relying solely on human review to catch AI-generated bugs

---

## Template 9: Feedback Survey

Use this monthly or quarterly survey to track AI tool adoption health.

### Template

```markdown
# AI Tools Feedback Survey — [Month/Quarter] [Year]
**Estimated time:** 5 minutes | **Anonymous:** Yes

## Section 1: Usage Frequency
1. How often do you use AI coding tools?
   - [ ] Multiple times daily
   - [ ] Once daily
   - [ ] A few times per week
   - [ ] Rarely (less than weekly)
   - [ ] Never

2. Which tools do you use? (Select all that apply)
   - [ ] GitHub Copilot (inline completions)
   - [ ] GitHub Copilot Chat
   - [ ] Claude (via API/web)
   - [ ] Cline (VS Code extension)
   - [ ] Cursor
   - [ ] Other: _____________

## Section 2: Effectiveness (1-5 Scale)
Rate each statement (1=Strongly Disagree, 5=Strongly Agree):

3. AI tools help me write code faster:          [1] [2] [3] [4] [5]
4. AI-generated code meets our quality standards: [1] [2] [3] [4] [5]
5. I feel confident reviewing AI-generated code:  [1] [2] [3] [4] [5]
6. Our instruction files improve AI output:       [1] [2] [3] [4] [5]
7. AI tools help me learn new patterns/APIs:      [1] [2] [3] [4] [5]
8. I understand our AI usage policy:              [1] [2] [3] [4] [5]

## Section 3: Pain Points
9. What is your biggest frustration with AI tools? (Select one)
   - [ ] Suggestions are irrelevant or low quality
   - [ ] Tools are slow or unreliable
   - [ ] I don't know how to use them effectively
   - [ ] They don't understand our codebase/patterns
   - [ ] Security/privacy concerns
   - [ ] Other: _____________

10. What task do you wish AI handled better?
    [Free text response]

## Section 4: Improvement Ideas
11. What would make AI tools more useful for you?
    [Free text response]

12. Would you recommend AI tools to a colleague?
    - [ ] Definitely yes
    - [ ] Probably yes
    - [ ] Neutral
    - [ ] Probably no
    - [ ] Definitely no

## Section 5: Open Feedback
13. Anything else you'd like to share about AI tools on our team?
    [Free text response]
```

### Customization Instructions

- Keep surveys short (under 5 minutes) to maximize response rates
- Run consistently (monthly is ideal during adoption, quarterly when mature)
- Share results with the team — transparency builds trust
- Act on feedback visibly — "You said X, we did Y"
- Use a proper survey tool (Google Forms, Typeform) rather than markdown

✅ **Good:** Acting on survey results and communicating changes
❌ **Bad:** Collecting feedback but never doing anything with it

---

## Template 10: Maturity Assessment

Use this quarterly self-assessment to track your team's AI adoption journey.

### Template

```markdown
# AI Adoption Maturity Assessment
**Team:** [Team Name] | **Date:** [Date] | **Assessor:** [Name]

## Scoring Guide
- **1 - Ad Hoc:** No formal approach; individuals do their own thing
- **2 - Emerging:** Some awareness; early experiments underway
- **3 - Defined:** Documented practices; consistent usage across team
- **4 - Managed:** Measured and monitored; data-driven improvements
- **5 - Optimizing:** Continuously improving; industry-leading practices

## Assessment Dimensions

### 1. Tool Adoption
| Criteria                                    | Score (1-5) | Evidence / Notes |
|---------------------------------------------|-------------|------------------|
| All team members have AI tools installed     | [  ]        |                  |
| Team members use AI tools daily              | [  ]        |                  |
| Multiple AI tools are used appropriately     | [  ]        |                  |
| Tools are configured with team settings      | [  ]        |                  |
| **Dimension Average**                        | **[  ]**    |                  |

### 2. Instruction Quality
| Criteria                                    | Score (1-5) | Evidence / Notes |
|---------------------------------------------|-------------|------------------|
| Instruction files exist and are maintained   | [  ]        |                  |
| Instructions produce consistent output       | [  ]        |                  |
| Instructions cover key coding standards      | [  ]        |                  |
| Team regularly updates instructions          | [  ]        |                  |
| **Dimension Average**                        | **[  ]**    |                  |

### 3. Review Process
| Criteria                                    | Score (1-5) | Evidence / Notes |
|---------------------------------------------|-------------|------------------|
| PRs disclose AI usage consistently           | [  ]        |                  |
| Reviewers know how to review AI code         | [  ]        |                  |
| Quality gates catch AI-specific issues       | [  ]        |                  |
| Review turnaround time is acceptable         | [  ]        |                  |
| **Dimension Average**                        | **[  ]**    |                  |

### 4. Governance & Policy
| Criteria                                    | Score (1-5) | Evidence / Notes |
|---------------------------------------------|-------------|------------------|
| AI usage policy exists and is known          | [  ]        |                  |
| Data classification is understood            | [  ]        |                  |
| AI Champion role is filled and active        | [  ]        |                  |
| Feedback mechanisms are in place             | [  ]        |                  |
| **Dimension Average**                        | **[  ]**    |                  |

### 5. Knowledge Sharing
| Criteria                                    | Score (1-5) | Evidence / Notes |
|---------------------------------------------|-------------|------------------|
| Team shares AI tips and prompts              | [  ]        |                  |
| Prompt library exists and is maintained      | [  ]        |                  |
| Cross-team AI knowledge sharing happens      | [  ]        |                  |
| New members are onboarded to AI workflow     | [  ]        |                  |
| **Dimension Average**                        | **[  ]**    |                  |

### 6. Measurement & ROI
| Criteria                                    | Score (1-5) | Evidence / Notes |
|---------------------------------------------|-------------|------------------|
| Baseline metrics were captured before AI     | [  ]        |                  |
| AI impact is regularly measured              | [  ]        |                  |
| ROI reports are produced and shared          | [  ]        |                  |
| Data drives adoption decisions               | [  ]        |                  |
| **Dimension Average**                        | **[  ]**    |                  |

## Summary

| Dimension             | Score | Target | Gap  |
|-----------------------|-------|--------|------|
| Tool Adoption         | [  ]  | [  ]   | [  ] |
| Instruction Quality   | [  ]  | [  ]   | [  ] |
| Review Process        | [  ]  | [  ]   | [  ] |
| Governance & Policy   | [  ]  | [  ]   | [  ] |
| Knowledge Sharing     | [  ]  | [  ]   | [  ] |
| Measurement & ROI     | [  ]  | [  ]   | [  ] |
| **Overall Average**   |**[  ]**|**[  ]**|**[  ]**|

## Maturity Level
Based on overall average:
- **1.0–1.9:** Level 1 — Ad Hoc (Focus: Get tools installed and policy written)
- **2.0–2.9:** Level 2 — Emerging (Focus: Standardize practices and train team)
- **3.0–3.9:** Level 3 — Defined (Focus: Measure impact and optimize workflows)
- **4.0–4.9:** Level 4 — Managed (Focus: Share across teams, drive org adoption)
- **5.0:**     Level 5 — Optimizing (Focus: Innovate, contribute back to community)

## Action Plan
| Priority | Action                              | Owner  | Target Date |
|----------|-------------------------------------|--------|-------------|
| High     | [Gap with largest impact]           | [Name] | [Date]      |
| Medium   | [Next priority gap]                 | [Name] | [Date]      |
| Low      | [Nice-to-have improvement]          | [Name] | [Date]      |

## Historical Trend
| Quarter   | Overall Score | Level      |
|-----------|---------------|------------|
| [Q1 2024] | [N.N]         | [Level N]  |
| [Q2 2024] | [N.N]         | [Level N]  |
| [Q3 2024] | [N.N]         | [Level N]  |
| [Current] | [N.N]         | [Level N]  |
```

### Customization Instructions

- Set realistic targets based on your starting point (don't aim for Level 5 immediately)
- Score honestly — the value is in tracking improvement, not in high scores
- Use evidence to justify scores, not gut feelings
- Share results with the team to build accountability
- Run every quarter and track the trend over time

---

## Quick Reference: Which Template When?

| Situation                          | Template(s) to Use              |
|------------------------------------|---------------------------------|
| Starting AI adoption               | #1 (Policy), #3 (Onboarding)   |
| Setting up a new repository        | #2 (Instructions), #8 (Gates)  |
| First AI-assisted sprint           | #4 (PR Template), #5 (Retro)   |
| Justifying AI investment           | #7 (ROI), #9 (Survey)          |
| Defining the AI Champion role      | #6 (Job Description)           |
| Quarterly review of AI adoption    | #10 (Maturity), #7 (ROI)       |
| Team struggling with adoption      | #9 (Survey), #10 (Maturity)    |
| Onboarding a new developer         | #3 (Onboarding), #1 (Policy)   |
