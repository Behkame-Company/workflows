# AI Adoption Maturity Model

> A five-stage framework for understanding where your team stands with AI tool adoption and what to focus on next. Progression is not about speed — it's about building a solid foundation at each stage.

---

## The Five Stages

```
┌─────────────────────────────────────────────────────────────┐
│  Stage 5: Innovation                                        │
│  Custom agents, multi-agent workflows, AI-native processes  │
├─────────────────────────────────────────────────────────────┤
│  Stage 4: Optimization                                      │
│  Metrics-driven, prompt library, custom skills              │
├─────────────────────────────────────────────────────────────┤
│  Stage 3: Standardization                                   │
│  Team-wide instruction files, quality gates, onboarding     │
├─────────────────────────────────────────────────────────────┤
│  Stage 2: Adoption                                          │
│  Agreed tools, basic instruction files, shared prompts      │
├─────────────────────────────────────────────────────────────┤
│  Stage 1: Experimentation                                   │
│  Individual exploration, no standards, mixed results        │
└─────────────────────────────────────────────────────────────┘
```

---

## Stage 1: Experimentation

**Characteristics:**
- Individual developers try AI tools on their own initiative
- No team agreement on which tools to use
- No shared instruction files or prompt standards
- Results vary wildly between developers
- AI usage is invisible in code reviews

**What you'll hear:**
- *"I've been trying Copilot, it's pretty good for boilerplate"*
- *"I just paste code into ChatGPT sometimes"*
- *"I don't trust AI code"*

**Risks at this stage:**
- Developers may share proprietary code with unapproved tools
- No guardrails against AI-generated security vulnerabilities
- Inconsistent and unreproducible results

**What to focus on:**
1. Identify who is using AI tools and which ones
2. Assess security implications of current tool usage
3. Select approved tools for team evaluation
4. Assign a champion to lead the next stage

**Exit criteria:** Team has agreed on which AI tools to evaluate.

---

## Stage 2: Adoption

**Characteristics:**
- Team agrees on one or two AI tools
- Basic instruction files exist (maybe one person created them)
- Some prompts are shared informally (Slack, meetings)
- AI tool setup is part of development environment
- Some developers are significantly more effective than others

**What you'll hear:**
- *"We're all using Copilot now"*
- *"Check out this prompt I found — it's great for writing tests"*
- *"The instruction file helps, but I still get inconsistent results"*

**Risks at this stage:**
- Instruction files may be incomplete or ignored
- Knowledge is shared informally and gets lost
- Quality variance is still high between developers

**What to focus on:**
1. Formalize the instruction file with team input
2. Start a shared prompt collection
3. Add AI-specific items to code review checklists
4. Begin documenting what works and what doesn't
5. Create a basic onboarding guide for new team members

**Exit criteria:** Team has a reviewed instruction file in version control and a documented review process.

---

## Stage 3: Standardization

**Characteristics:**
- Team-wide instruction files maintained through PRs
- Code review process includes AI-specific checks
- Quality gates enforce standards on AI output
- Onboarding guide exists for new developers
- Shared prompt library is actively maintained
- AI usage is visible and discussed in retrospectives

**What you'll hear:**
- *"The instruction file caught that — it knows we use the Result pattern"*
- *"Check the prompt library before writing that from scratch"*
- *"This PR was AI-assisted — I focused my review on the edge cases"*

**Risks at this stage:**
- Standards may become rigid and slow to evolve
- Measurement may focus on process compliance rather than outcomes
- Advanced users may feel constrained

**What to focus on:**
1. Establish metrics for AI-assisted development quality
2. Build feedback loops (retrospectives, surveys)
3. Begin developing custom prompts and skills
4. Cross-train team members on advanced patterns
5. Regular review and update of standards (quarterly)

**Exit criteria:** Team has measurable quality improvement and a self-sustaining improvement process.

---

## Stage 4: Optimization

**Characteristics:**
- Metrics track AI impact on velocity, quality, and developer satisfaction
- Prompt library is comprehensive and regularly updated
- Custom skills or agents exist for team-specific tasks
- Automated quality checks catch AI-specific issues
- AI tool configuration is optimized for team's tech stack
- Knowledge sharing is systematic, not ad-hoc

**What you'll hear:**
- *"Our AI-assisted PRs have 15% fewer bugs than last quarter"*
- *"I created a custom skill for our API boilerplate — try it out"*
- *"The automated check caught a hallucinated import"*

**Risks at this stage:**
- Over-optimization for metrics rather than real productivity
- Tool lock-in as custom tooling increases
- Maintenance burden of custom skills and agents

**What to focus on:**
1. Invest in custom tooling (agents, skills, MCP servers)
2. Share learnings across teams in the organization
3. Explore multi-agent workflows for complex tasks
4. Contribute back to tool ecosystems
5. Mentor other teams at earlier stages

**Exit criteria:** Team consistently measures and improves AI-assisted development outcomes.

---

## Stage 5: Innovation

**Characteristics:**
- Custom agents handle routine development tasks
- Multi-agent workflows for complex processes (test generation, migration, review)
- AI-native development processes (AI does first draft, humans refine)
- Team contributes back to AI tool ecosystems
- Continuous experimentation with emerging AI capabilities
- AI usage is natural and integrated, not a separate concern

**What you'll hear:**
- *"The migration agent handled 80% of the upgrade automatically"*
- *"Our review agent catches things human reviewers consistently miss"*
- *"We redesigned the workflow around AI capabilities"*

**Risks at this stage:**
- Over-reliance on AI tools (what happens when they're down?)
- Complexity of custom tooling becomes a maintenance burden
- Skills atrophy if developers stop understanding generated code

**What to focus on:**
1. Maintain human understanding of AI-generated systems
2. Build resilience (teams must function without AI tools)
3. Share innovation with the broader community
4. Continuously evaluate new AI capabilities
5. Mentor the next generation of AI-native developers

---

## Assessing Your Team's Stage

### Self-Assessment Checklist

Rate each statement (Yes / Partial / No):

**Stage 1 → 2 (Experimentation → Adoption)**
- [ ] Team has agreed on which AI tools to use
- [ ] At least one instruction file exists in the repository
- [ ] Developers know how to access and use the approved tools
- [ ] Basic security guidelines exist for AI tool usage

**Stage 2 → 3 (Adoption → Standardization)**
- [ ] Instruction files are maintained through PR review
- [ ] Code review process has AI-specific checkpoints
- [ ] Onboarding documentation covers AI tool setup and usage
- [ ] Team has a shared collection of prompts or patterns
- [ ] AI usage is discussed in team retrospectives

**Stage 3 → 4 (Standardization → Optimization)**
- [ ] Team tracks metrics related to AI-assisted development
- [ ] Prompt library has more than 20 tested entries
- [ ] At least one custom skill or agent exists
- [ ] Automated quality checks catch AI-specific issues
- [ ] Standards are reviewed and updated quarterly

**Stage 4 → 5 (Optimization → Innovation)**
- [ ] Custom agents handle recurring development tasks
- [ ] Multi-agent workflows exist for complex processes
- [ ] Team has redesigned at least one process around AI capabilities
- [ ] Team contributes to AI tool ecosystems (plugins, extensions, blog posts)
- [ ] New team members learn AI-native workflows from day one

### Scoring

- **Mostly "No"** → You're at the lower stage
- **Mix of "Partial" and "Yes"** → You're transitioning
- **Mostly "Yes"** → You're ready for the next stage

## Common Progression Mistakes

❌ **Skipping stages** — jumping to custom agents before having solid instruction files
❌ **Staying too long** — perfecting Stage 2 when you should move to Stage 3
❌ **Forcing progression** — mandating Stage 4 practices on a Stage 1 team
❌ **Measuring the wrong things** — counting AI usage instead of measuring outcomes
❌ **Ignoring regression** — teams can slip back if standards aren't maintained

✅ **Healthy progression** — spend 1-3 months per stage, validate exit criteria, then advance

## Next Steps

- [Understand the team lead's role in driving progression →](role-of-team-lead.md)
- [Build the cultural foundation for AI adoption →](ai-first-culture.md)
- [Start onboarding with a structured approach →](../02-onboarding/developer-onboarding.md)
- [Create the standards needed for Stage 3 →](../03-standards-and-conventions/instruction-file-standards.md)
