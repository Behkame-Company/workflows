# FAQ

> Frequently asked questions about AI code review and CI/CD integration.

---

## General

### Q: Does AI code review replace human reviewers?
**No.** AI handles mechanical checks (style, patterns, common bugs) so humans can focus on architecture, business logic, and design decisions. Always require at least one human approval.

### Q: How much does AI code review cost?
**GitHub Copilot**: Included with Copilot subscription ($10-39/user/month). **CodeRabbit**: Free for open source, Pro at $15/user/month. **PR-Agent**: Open source (self-hosted free), cloud from $19/month.

### Q: Which AI reviewer should I use?
Start with **GitHub Copilot code review** if you have a Copilot subscription — it's integrated into GitHub. Add **CodeRabbit** if you want a second opinion or more configurable rules.

### Q: How long does AI review take?
Typically **30 seconds to 3 minutes** depending on PR size. Much faster than waiting for a human reviewer.

---

## Setup

### Q: How do I enable Copilot code review?
Settings → Rules → New Ruleset → Add "Request Copilot code review" as a required check. See [Copilot Code Review](../02-review-tools/github-copilot-code-review.md).

### Q: Can I use AI review on private repos?
Yes, with a paid Copilot plan. Free plans may have limitations on private repos.

### Q: Does AI review work with monorepos?
Yes. Use path-specific instructions (`.instructions.md`) and path filters to handle different areas differently.

### Q: Can I use AI review with GitLab/Bitbucket?
CodeRabbit supports GitHub, GitLab, Bitbucket, and Azure DevOps. Copilot review is GitHub-only. PR-Agent supports multiple platforms.

---

## Quality

### Q: What's a good false positive rate?
**<20%** is the target. If more than 1 in 5 AI comments is wrong, developers will start ignoring all comments.

### Q: What's a good suggestion acceptance rate?
**30-60%** is healthy. Below 30% suggests too much noise. Above 60% is excellent.

### Q: How do I reduce false positives?
Write custom instructions that exclude known false positive patterns, tell AI what your linter covers, and use "chill" mode. See [Signal-to-Noise](../08-best-practices/signal-to-noise.md).

### Q: Can AI catch security vulnerabilities?
Yes, for common patterns (SQL injection, XSS, hardcoded secrets). For deep security analysis, use dedicated SAST tools (CodeQL, Semgrep) alongside AI review.

---

## CI/CD

### Q: Should AI review block merges?
**Eventually, yes** — but start non-blocking (shadow mode). Block only after tuning false positives. See [Incremental Adoption](../08-best-practices/incremental-adoption.md).

### Q: Can AI auto-fix CI failures?
Yes, for mechanical failures (lint, type errors, simple test fixes). Never auto-merge AI fixes. See [Self-Healing CI](../04-ai-in-ci-cd/self-healing-ci.md).

### Q: What about GitHub Agentic Workflows?
Agentic Workflows let you define CI/CD intent in Markdown and compile to GitHub Actions YAML. It's in technical preview. See [Agentic Workflows](../04-ai-in-ci-cd/agentic-workflows.md).

### Q: How do I integrate AI review into existing CI/CD?
Add AI review as a status check alongside your existing checks. It doesn't replace your pipeline — it adds a review step.

---

## Team

### Q: How do I get my team to adopt AI review?
Start in shadow mode (non-blocking), collect wins, share metrics, tune instructions based on feedback. See [Team Adoption](../07-human-ai-collaboration/team-adoption.md).

### Q: What if developers resist AI review?
Address specific objections: noise → tune instructions; slowness → show timing data; trust → start non-blocking. See [Common Objections](../07-human-ai-collaboration/team-adoption.md).

### Q: Should I track who dismisses AI findings?
Track patterns (which rule types get dismissed), not individuals. Use this to tune instructions, not to judge developers.

### Q: How often should I update review instructions?
**Every 2 weeks** during first 3 months, then **monthly**. Review dismissed findings to find tuning opportunities.
