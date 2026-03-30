# Frequently Asked Questions

> Quick answers to the most common questions about AI-assisted development on your team. Organized by category so you can find what you need fast. For deeper dives, follow the links to the relevant documentation.

---

## Getting Started

### Q1: What AI tools does our team use?

Check your team's AI Usage Policy for the official list. A typical approved stack includes:

| Tool             | Purpose                        | Access                     |
|------------------|--------------------------------|----------------------------|
| GitHub Copilot   | Code completion + chat in IDE  | Business license via org   |
| Claude           | Advanced reasoning + code gen  | Team license               |
| Cline            | Agentic coding in VS Code     | Pro license                |
| Cursor           | AI-first code editor           | Business license           |

✅ **Do:** Use tools from the approved list
❌ **Don't:** Sign up for unapproved AI tools with company data without checking with your AI Champion first

---

### Q2: How do I install and set up AI tools?

Follow the [Developer Onboarding Checklist](./templates.md#template-3-developer-onboarding-checklist) for step-by-step setup. The short version:

1. **GitHub Copilot:** Install the extension from VS Code marketplace → Sign in with your GitHub org account
2. **Claude:** Sign up at claude.ai or install the CLI → Enter your team license key
3. **Cline:** Install from VS Code marketplace → Configure your API provider
4. **Cursor:** Download from cursor.com → Sign in with your team account

If you hit issues, see [Troubleshooting: Instruction Files Aren't Loading](./troubleshooting.md#issue-1-instruction-files-arent-loading).

---

### Q3: Where are our instruction files?

Instruction files are stored in the repository root:

```
your-repo/
├── .github/
│   └── copilot-instructions.md    ← GitHub Copilot
├── .cursorrules                    ← Cursor (legacy, single file)
├── .cursor/
│   └── rules/                     ← Cursor (modern, multiple files)
├── .clinerules                     ← Cline
└── .claude/                        ← Claude Code
```

Every team member working in the repo automatically picks up these instructions. If you don't see them, clone the repo again or check if they're on a different branch.

---

### Q4: What if I've never used AI coding tools before?

That's completely fine. Here's your learning path:

1. **Day 1:** Install the tools and just watch the suggestions (don't accept anything yet)
2. **Day 2–3:** Start accepting simple completions (variable names, imports, one-liners)
3. **Week 1:** Try using chat for code explanations and questions
4. **Week 2:** Use AI to generate tests and documentation
5. **Week 3+:** Experiment with larger code generation and agentic workflows

Ask your onboarding buddy or AI Champion for a paired session — watching someone experienced use AI tools is the fastest way to learn.

---

### Q5: Do I have to use AI tools?

Generally no — AI tools are recommended, not mandated. However:

✅ **Expected:** Give AI tools a fair try during onboarding (at least 2-3 weeks)
✅ **Expected:** Follow team standards for AI-assisted PRs when you do use them
❌ **Not expected:** Use AI for every single line of code
❌ **Not expected:** Abandon your existing workflow entirely

If you find AI tools unhelpful for your specific work, discuss with your team lead. There are legitimate cases where AI adds more friction than value.

---

## Daily Use

### Q6: When should I use AI assistance?

AI excels at some tasks and struggles with others:

| ✅ Great for AI                    | ❌ Not great for AI                |
|-----------------------------------|-------------------------------------|
| Boilerplate and CRUD operations   | Novel algorithm design              |
| Test generation                   | Complex architectural decisions     |
| Code documentation                | Security-critical logic             |
| Refactoring to known patterns     | Domain-specific business rules      |
| Learning new APIs/frameworks      | Performance-critical hot paths      |
| Regex and string manipulation     | Debugging subtle race conditions    |
| Data transformations              | Understanding legacy "why" context  |
| Prototyping and exploration       | Regulatory compliance code          |

**Rule of thumb:** Use AI for the "what" and "how," but you decide the "why" and "whether."

---

### Q7: How do I write good prompts for code generation?

Follow the **SCOPE** method:

- **S**pecify the technology (language, framework, library)
- **C**ontext about the project (architecture, patterns, constraints)
- **O**utput format (function, class, module, test)
- **P**atterns to follow (reference existing code, link to examples)
- **E**dge cases to handle (errors, empty inputs, boundary conditions)

✅ **Good prompt:**
```
Write a TypeScript Express middleware that:
- Validates JWT tokens from the Authorization header
- Extracts user ID and role from the token payload
- Attaches { userId, role } to req.user
- Returns 401 for missing/invalid tokens with JSON error response
- Uses our existing JwtService from src/services/jwt.service.ts
```

❌ **Bad prompt:**
```
Write auth middleware
```

---

### Q8: What if AI generates bad or incorrect code?

This is normal and expected. AI-generated code is a first draft, not a final product.

1. **Don't panic.** Bad suggestions are part of the workflow
2. **Don't accept blindly.** Read every line before accepting
3. **Iterate.** Refine your prompt, add context, try again
4. **Know when to stop.** If AI can't get it right after 2-3 attempts, write it yourself
5. **Report patterns.** If AI consistently gets something wrong, update the instruction file

✅ **Good:** "AI suggested string concatenation for SQL — I caught it and used parameterized queries"
❌ **Bad:** "AI wrote it so it must be fine" → accepts without review → security vulnerability

---

### Q9: How do I use AI for code I don't fully understand?

AI is a great learning tool, but be cautious:

✅ **Safe approaches:**
- Ask AI to explain code: "Explain what this function does line by line"
- Ask for simplified versions: "Rewrite this using simpler patterns"
- Use AI to add comments: "Add inline comments explaining the logic"
- Ask about alternatives: "What are other ways to implement this?"

❌ **Risky approaches:**
- Copying AI-generated code you don't understand into production
- Using AI as a substitute for learning core concepts
- Trusting AI explanations without verifying (AI can confidently explain wrong things)

---

### Q10: Can I use AI to write commit messages?

Yes, with review:

✅ **Good workflow:** Let AI draft the commit message, then edit it to be accurate
❌ **Bad workflow:** Accept AI-generated commit messages without reading them

AI often produces generic messages. Make sure your commit messages accurately describe WHAT changed and WHY.

---

### Q11: How do I use AI when pair programming?

AI adds a "third voice" to pair programming:

- **Navigator can ask AI** for alternative approaches while driver codes
- **Driver can use AI completions** for boilerplate while navigator reviews
- **Both discuss AI suggestions** as a learning exercise
- **Avoid:** One person driving while AI generates and the other person disengages

---

## Review Process

### Q12: How do I review AI-generated code?

Review AI code with EXTRA scrutiny, not less. Focus on:

1. **Logic correctness:** Does it actually do what it's supposed to? (AI often gets the happy path right but misses edge cases)
2. **Error handling:** Are errors caught and handled according to team standards?
3. **Security:** Check for injection, hardcoded secrets, insecure defaults
4. **Performance:** AI doesn't optimize for your specific scale
5. **Maintainability:** Is the code readable? Would a new team member understand it?
6. **Necessity:** Is all the generated code needed? AI tends to over-generate

✅ **Good review comment:** "This AI-generated validation misses the case where email contains unicode characters"
❌ **Bad review comment:** "AI-generated code, LGTM" (rubber-stamping)

---

### Q13: What's our PR template for AI-assisted code?

See the [PR Review Template](./templates.md#template-4-pr-review-template-ai-disclosure). Key sections:

1. **AI Disclosure:** Which tools were used and for what
2. **Estimated AI contribution:** Low/Medium/High
3. **What AI generated vs. what you modified:** Be specific
4. **Review focus areas:** Guide reviewers to areas needing attention

---

### Q14: Do I need to disclose AI use in PRs?

Yes. Our PR template includes an AI disclosure section. This isn't about blame — it's about:

- Helping reviewers know where to focus their attention
- Building team transparency and trust
- Tracking AI adoption metrics over time
- Meeting potential compliance or regulatory requirements

✅ **Do:** Fill out the AI disclosure section honestly
❌ **Don't:** Hide AI use or exaggerate human contribution

---

### Q15: How many reviewers are needed for AI-assisted PRs?

Follow your team's standard review policy. Common approaches:

| AI Involvement | Recommended Reviews |
|----------------|---------------------|
| Low (completion only) | Standard (1 review) |
| Medium (chat-generated functions) | Standard (1 review with focus) |
| High (agent-generated features) | Enhanced (2 reviews) |
| Security-sensitive | Enhanced + security team review |

---

### Q16: What if I disagree with an AI-generated approach in a PR?

Treat it like any other code review disagreement:

1. Comment with your concern and a suggested alternative
2. Discuss with the author (the human, not the AI)
3. Remember: the author owns the code regardless of how it was generated
4. If unresolved, escalate to tech lead

---

## Security & Data

### Q17: What data can I share with AI tools?

Follow your organization's data classification:

| Classification | Can Share? | Examples                                    |
|----------------|------------|---------------------------------------------|
| Public         | ✅ Yes     | Open-source code, public docs, READMEs      |
| Internal       | ✅ Yes     | Internal code, business logic, architecture  |
| Confidential   | ⚠️ Caution | Customer data structures (anonymize first)   |
| Restricted     | ❌ Never   | PII, passwords, API keys, tokens, secrets    |

**When in doubt, don't share it.** Ask your AI Champion or security team.

---

### Q18: Where is my data stored when I use AI tools?

It depends on the tool and your organization's license:

| Tool           | Business/Team Plan                    | Individual Plan                |
|----------------|---------------------------------------|--------------------------------|
| GitHub Copilot | Not stored or used for training       | Check current ToS              |
| Claude         | Not used for training (Team/API)      | May be used for training       |
| Cursor         | Privacy mode available                | Check current ToS              |
| Cline          | Depends on API provider configured    | Depends on API provider        |

✅ **Use:** Organization/Business/Team tier licenses
❌ **Avoid:** Personal free tier accounts for company code

Always verify your organization's specific agreement with each vendor.

---

### Q19: What if AI suggests code with a known vulnerability?

1. **Don't accept it.** Reject the suggestion
2. **Fix it.** Write the secure version yourself or prompt for a secure alternative
3. **Report it.** Note it in your team's AI feedback channel so others are aware
4. **Update instructions.** Add a rule to the instruction file to prevent recurrence

Example:
```
❌ AI suggested:  db.query("SELECT * FROM users WHERE id = " + userId)
✅ Correct:       db.query("SELECT * FROM users WHERE id = $1", [userId])
```

---

### Q20: Can I paste error messages and stack traces into AI chat?

Generally yes, but scrub sensitive data first:

✅ **Safe to share:** Error types, stack traces, line numbers, generic messages
❌ **Not safe:** Errors containing PII, connection strings, API keys, passwords

**Quick check before pasting:** Search the error text for `@`, `://`, `key`, `secret`, `password`, `token`. If any are present, redact those portions.

---

### Q21: Is AI-generated code subject to copyright issues?

This is an evolving legal area. Current best practices:

- AI-generated code is generally treated as authored by the developer using the tool
- Do NOT ask AI to reproduce specific copyrighted code verbatim
- If AI output looks suspiciously like a specific library's implementation, verify its origin
- Follow your organization's legal guidance on AI-generated IP
- GitHub Copilot Business includes an IP indemnity feature — verify it's enabled

---

## Governance & Policy

### Q22: Who decides our AI policy?

Typically a collaboration between:

| Stakeholder       | Responsibility                                    |
|-------------------|---------------------------------------------------|
| Engineering Lead  | Technical standards and tool selection             |
| AI Champion       | Day-to-day policy implementation and feedback      |
| Security Team     | Data classification and security requirements      |
| Legal/Compliance  | Licensing, IP, regulatory requirements             |
| Engineering Mgmt  | Budget, adoption strategy, resource allocation     |

The AI Usage Policy (see [Template 1](./templates.md#template-1-ai-usage-policy-one-page)) should be reviewed and updated quarterly.

---

### Q23: How do I request a new AI tool?

1. **Identify the need:** What does the new tool do that approved tools don't?
2. **Evaluate:** Does it meet security and data classification requirements?
3. **Propose:** Submit a request to your AI Champion or engineering manager
4. **Trial:** If approved, run a time-boxed pilot with a small group
5. **Decide:** Based on pilot results, approve or reject for wider use

✅ **Good request:** "Cursor's multi-file editing would save our frontend team 3+ hours/week on component refactoring. It meets our security requirements because..."
❌ **Bad request:** "I saw a cool new AI tool on Twitter, can we use it?"

---

### Q24: What's our AI usage policy?

Your team should have a one-page policy (see [Template 1](./templates.md#template-1-ai-usage-policy-one-page)). Key points:

1. You own and are responsible for all AI-generated code
2. Never share restricted data (secrets, PII) with AI tools
3. Review all AI output before committing
4. Disclose AI use in PRs
5. Follow existing coding standards — AI code isn't exempt

---

### Q25: How often should we update our AI policy?

- **Quarterly:** Review and update the AI Usage Policy
- **Monthly:** Review and update instruction files
- **Per sprint:** Review AI section in retrospective
- **As needed:** When new tools are adopted, security incidents occur, or regulations change

---

### Q26: Do we need different policies for different teams?

Use a two-layer approach:

- **Organization-wide:** Approved tools, security rules, data classification (non-negotiable)
- **Team-specific:** Instruction files, prompt libraries, workflow customizations (flexible)

This balances consistency with team autonomy.

---

## Advanced Usage

### Q27: How do I create custom instructions or skills?

Depending on your tool:

**GitHub Copilot:**
- Create `.github/copilot-instructions.md` in your repo
- See [Template 2](./templates.md#template-2-team-instruction-file-copilot-instructionsmd-starter) for a starter

**Cursor:**
- Create `.cursor/rules/` directory with multiple rule files
- Rules can be scoped to specific file patterns

**Cline:**
- Create `.clinerules` in your repo root
- Can include project-specific context and conventions

**Claude Code:**
- Create `.claude/` directory with configuration files
- Supports CLAUDE.md project-level instructions

---

### Q28: Can I use MCP (Model Context Protocol) servers?

MCP servers extend AI tool capabilities by connecting to external data sources. Check with your team:

✅ **Approved MCP servers:** Your team should maintain a list of approved servers
❌ **Don't:** Connect random MCP servers from the internet without security review

Common useful MCP servers:
- **Database:** Query your development database from AI chat
- **Documentation:** Pull internal docs into AI context
- **Issue tracker:** Reference Jira/Linear tickets in AI conversations

Talk to your AI Champion about which MCP servers are available and approved.

---

### Q29: How do I contribute to the team prompt library?

Most teams maintain a shared prompt library. To contribute:

1. **Document your prompt:** Include the prompt text, when to use it, and example output
2. **Test it:** Verify it works consistently for your use case
3. **Submit it:** Add it to the prompt library via PR (location varies by team)
4. **Categorize it:** Use consistent categories (testing, refactoring, documentation, etc.)

✅ **Good prompt contribution:**
```markdown
## Generate Unit Tests for React Components
**When to use:** After creating a new React component
**Prompt:**
"Write Jest + React Testing Library tests for this component.
Test: rendering, user interactions, error states, loading states.
Follow our test patterns in src/__tests__/examples/"
**Example output:** [link to example]
```

❌ **Bad prompt contribution:** "Test prompt: write tests" (too vague to be reusable)

---

### Q30: How do I handle AI tools in monorepos?

Monorepos need special consideration:

- **Instruction files:** Place at repo root; consider tool-specific scoping (e.g., Cursor rules per directory)
- **Context:** AI may pull context from unrelated packages — be explicit about which package you're working in
- **Prompts:** Always specify the package/module: "In the `packages/auth` module, create..."

---

### Q31: Can I use AI for database migrations?

With extreme caution:

✅ **Safe:** Ask AI to draft a migration, then review carefully before running
✅ **Safe:** Use AI to generate the rollback migration after writing the forward migration
❌ **Dangerous:** Letting AI generate and run migrations without review
❌ **Dangerous:** Using AI for data migrations on production data

Always test migrations on a development database first, regardless of how they were generated.

---

### Q32: How should I use AI for debugging?

AI is excellent for debugging when used correctly:

1. **Provide context:** Paste the error message, relevant code, and what you've tried
2. **Be specific:** "This function returns null instead of the user object when the email contains a '+' character"
3. **Verify fixes:** Don't blindly apply AI's debugging suggestion — understand WHY it works
4. **Use breakpoints:** Let AI suggest where to add breakpoints, then inspect yourself

✅ **Good:** "Here's the error, the function, and the test case. Why does it fail for this input?"
❌ **Bad:** "My code doesn't work, fix it" (no context)

---

### Q33: How do I use AI for code reviews?

AI can assist (but not replace) human code review:

- **Pre-review:** Ask AI to analyze a diff for potential issues before you start
- **Understanding:** Ask AI to explain complex changes you're reviewing
- **Standards checking:** Use AI to verify code follows team conventions
- **Test coverage:** Ask AI to identify untested code paths

✅ **Good:** Use AI to prepare for your review, then apply your own judgment
❌ **Bad:** Let AI write your review comments without human verification

---

### Q34: What's the best way to learn AI-assisted development?

Progressive learning path:

1. **Watch:** Observe an experienced team member's AI workflow (1 hour)
2. **Copy:** Follow their patterns with your own tasks (1 week)
3. **Experiment:** Try different prompting strategies (2 weeks)
4. **Optimize:** Develop your own efficiency patterns (ongoing)
5. **Share:** Teach others what you've learned (ongoing)

Resources: See [Resources & Further Reading](./resources.md) for courses, talks, and guides.

---

### Q35: How do I handle AI hallucinations (confidently wrong output)?

AI can generate plausible-looking code that's completely wrong. Watch for:

- **Fake APIs:** AI invents function names or library methods that don't exist
- **Wrong versions:** AI suggests syntax from an older/newer version of a library
- **Incorrect logic:** Code that looks right but has subtle logical errors
- **Made-up patterns:** AI combines real patterns into something that doesn't work

**Defense strategies:**
1. Always compile/run AI-generated code before committing
2. Check library documentation when AI references unfamiliar APIs
3. Write tests that verify behavior, not just that code runs
4. Be skeptical of AI confidence — it's always confident, even when wrong

---

### Q36: Should I use AI differently for frontend vs. backend code?

Some adjustments help:

**Frontend:**
- AI is great for component boilerplate, CSS, and layout code
- Be cautious with accessibility — AI often generates inaccessible markup
- Verify responsive behavior manually (AI can't test visual output)
- Use AI for design system component scaffolding

**Backend:**
- Focus on security review for AI-generated API endpoints
- Verify database query performance (AI doesn't know your data volume)
- Double-check error handling and logging patterns
- Be cautious with AI-generated authentication/authorization logic

---

### Q37: How do I use AI tools offline or in restricted environments?

Options for air-gapped or restricted environments:

- **GitHub Copilot:** Requires internet connection (no offline mode)
- **Cursor:** Requires internet connection
- **Cline:** Can work with local models via Ollama or LM Studio
- **Claude:** API requires internet; no offline option

For restricted environments, discuss with your security team about approved network configurations or on-premises AI model deployments.

---

### Q38: Can AI help with technical debt reduction?

Yes — AI is excellent for mechanical refactoring:

✅ **Good AI tasks for tech debt:**
- Updating deprecated API usage across the codebase
- Adding types to untyped JavaScript/Python code
- Extracting duplicated code into shared utilities
- Adding missing error handling to existing functions
- Generating tests for untested legacy code
- Updating code to match new coding standards

❌ **Not great for AI:**
- Deciding WHAT technical debt to prioritize
- Architectural refactoring (changing fundamental patterns)
- Understanding why legacy code was written a certain way

---

### Q39: How do I use AI effectively in a new codebase I don't know?

AI is a fantastic exploration tool:

1. **Ask for overview:** "Explain the architecture of this project based on the directory structure"
2. **Trace flows:** "Walk me through what happens when a user logs in"
3. **Understand patterns:** "What design patterns does this codebase use?"
4. **Find entry points:** "Where are the main API route definitions?"

But remember: AI might hallucinate details about the codebase. Verify important claims by reading the actual code.

---

### Q40: What's the difference between Copilot, Claude, Cline, and Cursor?

| Feature                  | Copilot       | Claude         | Cline          | Cursor        |
|--------------------------|---------------|----------------|----------------|---------------|
| **Primary mode**         | Completions   | Chat/reasoning | Agentic        | AI-first IDE  |
| **IDE integration**      | VS Code + JetBrains | Web + API + CLI | VS Code    | Own editor    |
| **Best at**              | Inline suggestions | Complex reasoning | Multi-file changes | Rapid prototyping |
| **Context awareness**    | Current file + open tabs | Conversation window | Full project | Full project |
| **Learning curve**       | Low           | Medium         | Medium         | Medium        |

Most teams use 2-3 of these in combination rather than picking just one.

---

## Workflow Integration

### Q41: How does AI fit into our sprint workflow?

Integrate AI naturally into existing ceremonies:

- **Sprint Planning:** Consider AI assistance when estimating effort
- **Daily Standup:** Mention AI-related blockers or learnings
- **Sprint Review:** Demonstrate AI-assisted features (same as any other)
- **Retrospective:** Include AI section (see [Template 5](./templates.md#template-5-sprint-retrospective-guide-ai-section))

---

### Q42: Should we estimate differently for AI-assisted work?

Initially, no. As you gather data:

1. **Track actuals:** How long did AI-assisted tasks take vs. estimates?
2. **Adjust gradually:** If AI consistently halves boilerplate time, factor that in
3. **Don't over-promise:** AI speeds up some tasks but not all
4. **Account for review:** AI-generated code still needs thorough review

---

### Q43: How do we handle AI tool outages?

AI tools are cloud-dependent. Plan for outages:

1. **Don't block on AI:** Never make AI tools a hard dependency for your workflow
2. **Have fallback plans:** Know how to do your work without AI assistance
3. **Monitor status pages:** Bookmark status.github.com, status.anthropic.com, etc.
4. **Cache what you can:** Keep local copies of frequently used prompts and patterns

---

### Q44: Can I use AI tools in production incident response?

With care:

✅ **Good uses during incidents:**
- Quickly understanding unfamiliar code paths
- Generating database queries to investigate issues
- Drafting incident communication templates
- Analyzing log patterns

❌ **Bad uses during incidents:**
- Generating and deploying production fixes without review
- Relying on AI to diagnose root causes without verification
- Sharing production secrets or customer data with AI tools

---

### Q45: How do AI tools work with feature flags?

AI can help with feature flag implementation:

- **Generating flag configurations:** "Create a feature flag for the new checkout flow"
- **Adding flag checks:** "Wrap this feature in a feature flag using our LaunchDarkly setup"
- **Cleanup:** "Remove the feature flag for [feature] and delete the old code path"

Make sure AI follows your team's feature flag naming conventions and patterns.

---

## Troubleshooting Quick Reference

### Q46: My Copilot suggestions are terrible. What do I do?

Quick checklist:
1. Is the instruction file loading? (See [Issue 1](./troubleshooting.md#issue-1-instruction-files-arent-loading))
2. Do you have relevant files open for context?
3. Is your prompt specific enough? (See [Q7](#q7-how-do-i-write-good-prompts-for-code-generation))
4. Have you tried restarting the extension?
5. Check the Copilot status bar icon — is it connected?

---

### Q47: AI keeps generating code in the wrong language/framework.

Your instruction file or prompt context may be ambiguous:

1. Explicitly state the language in your prompt: "In TypeScript using Express..."
2. Make sure the instruction file specifies your tech stack
3. Ensure your file extension matches the expected language
4. Close files from other projects that might confuse the context

---

### Q48: How do I report a bug in an AI tool?

| Tool           | Bug Report Channel                                |
|----------------|---------------------------------------------------|
| GitHub Copilot | github.com/community → Copilot category           |
| Claude         | support.anthropic.com                              |
| Cline          | github.com/cline/cline → Issues                   |
| Cursor         | forum.cursor.com or in-app feedback                |

Always include: tool version, editor version, OS, and steps to reproduce.

---

## Team Culture

### Q49: How do we prevent AI from creating a "two-tier" team?

Risk: developers who use AI become "fast" while others feel left behind.

**Prevention strategies:**
1. Invest equally in training for all team members
2. Don't compare individual productivity metrics
3. Celebrate human skills that AI can't replace (architecture, mentoring, creativity)
4. Make AI knowledge sharing part of the team culture
5. Pair experienced AI users with newcomers

---

### Q50: How do we maintain code ownership when AI generates code?

Simple rule: **whoever commits the code owns it.**

- The human author is responsible for understanding, reviewing, and maintaining AI-generated code
- "AI wrote it" is never an excuse for bugs, security issues, or poor quality
- Code ownership tracking (CODEOWNERS) applies to AI-generated code the same as human-written code

✅ **Good:** "I used AI to generate the initial structure, then customized the error handling and added edge case tests"
❌ **Bad:** "I don't know how this works — AI generated it"
