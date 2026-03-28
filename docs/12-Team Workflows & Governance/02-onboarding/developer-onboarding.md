# Developer Onboarding Guide

> Getting new team members productive with AI tools through a structured, phased approach. The key prerequisite: your team must HAVE standards before you can onboard anyone to them.

---

## Prerequisites

Before onboarding a new developer to AI workflows, ensure your team has:

- [ ] An approved set of AI tools with licenses available
- [ ] A shared instruction file in the repository (`copilot-instructions.md`, `CLAUDE.md`)
- [ ] A documented code review process for AI-generated code
- [ ] At least a starter prompt library
- [ ] An assigned onboarding buddy (experienced AI user)

> **No standards yet?** Start with [Instruction File Standards](../03-standards-and-conventions/instruction-file-standards.md) first. Onboarding to chaos isn't onboarding — it's hazing.

---

## Phase 1: Setup (Day 1)

**Goal:** Developer has all tools installed, configured, and verified.

### Checklist

- [ ] Install approved AI tools (see [Tool Setup Checklist](tool-setup-checklist.md))
- [ ] Authenticate and verify access
- [ ] Configure settings to match team standards
- [ ] Verify instruction files load correctly
- [ ] Run test prompts to confirm everything works
- [ ] Read the team instruction file top to bottom

### Verification

The new developer should be able to:
```
✅ Open their editor and see AI autocomplete working
✅ Run a CLI AI tool and get a response
✅ Confirm the instruction file is being read (ask the AI about team conventions)
✅ Access the team prompt library
```

### Day 1 Schedule

| Time | Activity | With |
|---|---|---|
| 9:00 | Welcome, overview of AI workflow | Team lead |
| 9:30 | Tool installation and configuration | Onboarding buddy |
| 11:00 | Read instruction files and prompt library | Self-guided |
| 12:00 | Lunch | |
| 13:00 | Guided first AI interaction | Onboarding buddy |
| 14:00 | Explore autocomplete with simple tasks | Self-guided |
| 15:00 | Try 3 prompts from the prompt library | Self-guided |
| 16:00 | Q&A and day 1 retro | Onboarding buddy |

---

## Phase 2: Guided Learning (Week 1)

**Goal:** Developer understands AI tool patterns and team standards through hands-on practice.

### Day 2: Instruction Files and Context

- Understand how instruction files shape AI behavior
- Experiment: same prompt with and without instruction file
- Learn context management (what to include, what to exclude)
- Try prompt patterns from the team library

**Exercise:** Generate a function using a prompt library template. Compare the output with and without the instruction file loaded.

### Day 3: Pair Programming with AI

- Shadow onboarding buddy during an AI-assisted task
- Learn when AI helps vs. when it hinders
- Practice the review-and-refine cycle
- Understand the difference between accepting and adapting AI output

**Exercise:** Pair with buddy on a real task. Buddy drives first, then new developer drives.

### Day 4: Testing with AI

- Learn TDD workflow with AI
- Generate tests for existing code
- Understand test quality assessment
- Learn verification patterns (boundary testing, error paths)

**Exercise:** Pick an existing function, generate tests with AI, evaluate coverage, and identify gaps.

### Day 5: Independent Task and Retrospective

- Complete a small, well-defined task using AI tools independently
- Submit a PR following team AI review process
- Review with onboarding buddy
- Week 1 retrospective

**Exercise:** Implement a feature from the backlog using AI assistance. Focus on process, not speed.

### Week 1 Daily Exercise List

| Day | Exercise | Expected outcome |
|---|---|---|
| 1 | Run 3 prompt library templates | Working code from shared prompts |
| 2 | Compare output with/without instruction file | Understanding of context impact |
| 3 | Pair program an AI-assisted feature | Experience with review-and-refine cycle |
| 4 | Generate and evaluate test suite | Understanding of AI test quality |
| 5 | Solo AI-assisted PR | Complete PR following team process |

---

## Phase 3: Independent Practice (Month 1)

**Goal:** Developer uses AI tools independently with appropriate judgment.

### Week 2-4 Activities

- Use AI tools for daily development work
- Receive focused feedback on AI-related aspects during PR reviews
- Contribute at least one prompt to the team library
- Identify one improvement for the instruction file
- Practice progressively more complex AI-assisted tasks

### Skill Progression

```
Week 1: Simple code generation, test writing
Week 2: Refactoring, debugging assistance, documentation
Week 3: Complex features, multi-file changes, architecture discussion
Week 4: Workflow optimization, custom prompts, mentoring ability
```

### Monthly Checkpoint

At the end of Month 1, the developer and their buddy assess:

- [ ] Can independently use AI tools for common development tasks
- [ ] Knows when to use AI and when not to
- [ ] Reviews AI output critically (doesn't accept blindly)
- [ ] Follows team AI review process consistently
- [ ] Has contributed to the prompt library or instruction file
- [ ] Can explain team AI standards to another person

---

## Phase 4: Ongoing Growth

**Goal:** Developer continuously improves their AI-assisted development skills and helps others.

### Activities

- Explore advanced patterns (multi-step prompts, custom skills)
- Experiment with new AI tool features as they're released
- Mentor newer team members on AI workflows
- Contribute regularly to prompt library and instruction files
- Share learnings in team meetings and channels
- Participate in instruction file quarterly reviews

### Growth Path

```
Month 1-3:  Proficient user — follows team standards, works independently
Month 3-6:  Advanced user — creates prompts, improves processes
Month 6-12: Champion — mentors others, drives innovation
Month 12+:  Expert — designs workflows, builds custom tools
```

---

## Onboarding Anti-Patterns

❌ **"Here's Copilot, figure it out"**
No structure, no standards, no support. The developer either struggles or develops bad habits.

❌ **"Watch this 2-hour video"**
Passive learning doesn't build skills. Hands-on practice with feedback is essential.

❌ **"Use AI for everything this week"**
Forcing AI usage for every task leads to frustration. Let developers use judgment about when AI helps.

❌ **"Don't worry about the instruction file"**
Skipping standards from day one means bad habits form immediately.

❌ **"Just ask the AI how to use AI"**
Meta-prompting without context isn't effective. Team-specific guidance beats generic AI advice.

✅ **"Here's our structured program with a buddy, standards, and exercises"**
Guided practice with feedback and team context.

---

## Onboarding Feedback Template

Collect this feedback at the end of Week 1 and Month 1:

```markdown
## AI Onboarding Feedback

### What worked well?
<!-- What aspects of the onboarding were most helpful? -->

### What was confusing?
<!-- What wasn't clear or could be explained better? -->

### What's missing?
<!-- What do you wish had been covered? -->

### Tool feedback
<!-- Any issues with tool setup, performance, or configuration? -->

### Standard feedback
<!-- Do the instruction files and standards make sense? Anything to add? -->

### Confidence level (1-5)
<!-- How confident do you feel using AI tools independently? -->

### Suggestions
<!-- How would you improve the onboarding for the next person? -->
```

## Next Steps

- [Complete the tool setup checklist →](tool-setup-checklist.md)
- [Follow the first week learning path →](first-week.md)
- [Avoid common beginner mistakes →](beginner-mistakes.md)
- [Explore the team prompt library →](../03-standards-and-conventions/prompt-library.md)
