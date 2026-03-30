# AI Retrospectives

> Learn from your team's AI usage through structured reflection — celebrate wins, analyze failures, and continuously improve how you work with AI tools.

---

## Why AI-Specific Retrospectives?

Standard sprint retrospectives rarely surface AI-specific insights. Developers may mention "AI helped" or "AI got in the way," but without dedicated reflection, teams miss the deeper patterns:

- Which AI workflows actually save time vs. which feel productive but aren't?
- Are certain types of tasks consistently better (or worse) with AI?
- Is the team developing bad habits (blindly accepting AI output)?
- Are there security or quality concerns that need addressing?

AI retrospectives create a structured space for these conversations.

### The Feedback Loop

```
    ┌──────────────────────────────────┐
    │                                  │
    ▼                                  │
┌─────────┐    ┌──────────┐    ┌──────┴──────┐
│  Use AI  │───▶│  Observe  │───▶│  Retrospect  │
│  in work │    │  results  │    │  as team     │
└─────────┘    └──────────┘    └──────┬──────┘
    ▲                                  │
    │          ┌──────────┐            │
    └──────────│  Improve  │◀───────────┘
               │  practices│
               └──────────┘
```

Without retrospectives, you're stuck in "Use AI → Observe" without ever reaching "Improve."

✅ **Good feedback loop:** Team discusses AI patterns every 2 weeks; practices visibly improve each month
❌ **Bad feedback loop:** Individual developers silently form opinions about AI; no shared learning occurs

---

## Regular Retrospective Format

### Core Questions

Every AI retrospective should address these five questions:

| # | Question | Purpose |
|---|----------|---------|
| 1 | What did AI help with this sprint? | Celebrate wins and identify high-value use cases |
| 2 | What didn't work with AI? | Surface friction points and failures |
| 3 | What should we try next? | Generate experiments for the next cycle |
| 4 | What prompts or patterns were most effective? | Capture and share successful techniques |
| 5 | Any security or quality concerns? | Surface risks before they become incidents |

### Extended Question Set

For deeper retrospectives (quarterly or when significant changes have occurred):

**Productivity**
- How much time did AI save us this sprint? (estimate)
- Were there tasks where AI *cost* us time? (rework, debugging AI output)
- What's our biggest remaining manual bottleneck that AI could address?

**Quality**
- Did any AI-generated code cause bugs in production?
- How thorough are our reviews of AI-generated code?
- Are we writing fewer tests because AI makes us overconfident?

**Learning**
- What new AI techniques did anyone discover this sprint?
- Are junior developers learning or just copying AI output?
- What did we learn about our codebase from how AI interacts with it?

**Culture**
- Is anyone uncomfortable with our level of AI usage?
- Are we over-relying on AI for any particular task?
- Is AI usage evenly distributed or concentrated in a few people?

✅ **Good questions:** Specific, answerable, lead to concrete actions
❌ **Bad questions:** "How do we feel about AI?" (too vague to produce actionable insights)

---

## AI-Specific Retro Categories

Structure your retrospective board around these five categories:

### 1. 🏆 Wins — Big Productivity Gains

Celebrate moments where AI made a real difference.

#### Example Win Entries

```
🏆 WIN: Used Copilot Chat to generate 40 unit tests for our
payment module in 2 hours. Manually would have taken 2 days.
Tests caught 3 bugs we didn't know about.
— @sarah

🏆 WIN: Custom instruction file for our API conventions means
Copilot now generates endpoints that pass linting on first try.
Setup cost: 30 minutes. Time saved per endpoint: 15 minutes.
— @mike

🏆 WIN: AI-assisted code review caught a SQL injection vector
that 3 human reviewers missed. Added the pattern to our
security review prompt.
— @alex
```

**Action from wins:** Document the prompts/patterns. Add to the shared prompt library.

---

### 2. 💥 Fails — AI Led Us Astray

Honest discussion of where AI hurt more than helped.

#### Example Fail Entries

```
💥 FAIL: Copilot confidently generated a caching implementation
that looked correct but had a subtle race condition. We
deployed it and it caused intermittent failures for 3 days
before we traced it back.
Lesson: Extra scrutiny for concurrent code.
— @bob

💥 FAIL: Spent 4 hours trying to get AI to refactor our legacy
auth module. The context window couldn't handle the complexity.
Would have been faster to do it manually from the start.
Lesson: Know when to stop prompt-engineering and just code.
— @carol

💥 FAIL: New developer accepted AI suggestions for database
queries without understanding them. Produced queries that
worked but were O(n²) instead of O(n).
Lesson: Junior devs need explicit guidance on when to question AI.
— @techlead
```

**Action from fails:** Update conventions, add to troubleshooting guide, create training materials.

✅ **Good fail discussion:** Blame-free, focused on systemic improvements, specific about what went wrong
❌ **Bad fail discussion:** "AI is unreliable" (too generic) or blaming individuals for trusting AI

---

### 3. 😲 Surprises — Unexpected AI Behaviors

Document unexpected behaviors — both positive and negative.

#### Example Surprise Entries

```
😲 SURPRISE: Copilot started suggesting TypeScript code in our
Python files after a VS Code update. Turns out it was reading
context from a different open tab. Closing unrelated tabs fixed it.
— @sarah

😲 SURPRISE: Asked AI to "simplify this function" and it
completely rewrote the algorithm with a 3x performance improvement
we hadn't considered. The approach was valid and we kept it.
— @mike

😲 SURPRISE: AI suggestions changed noticeably after the model
update last Tuesday. Some of our standard prompts stopped
working as well. Need to re-test the prompt library.
— @alex
```

**Action from surprises:** Investigate root causes, update documentation, adjust expectations.

---

### 4. 📚 Learnings — New Techniques Discovered

Share techniques that team members discovered during the sprint.

#### Example Learning Entries

```
📚 LEARNING: Breaking large prompts into a sequence of smaller,
focused prompts produces much better results than one giant
prompt. Applied this to our API generation workflow.
— @bob

📚 LEARNING: Including 2-3 examples of the desired output format
in the prompt dramatically improves consistency. Our test
generation prompt went from 60% to 95% format compliance.
— @carol

📚 LEARNING: Copilot Chat with @workspace is really effective
for understanding unfamiliar parts of our codebase. Faster
than grep-ing through files for new team members.
— @newdev
```

**Action from learnings:** Add to prompt library, share in Slack, include in onboarding.

---

### 5. 🎯 Actions — Changes to Make

Concrete changes the team commits to for the next sprint.

#### Example Action Items

```
🎯 ACTION: Add "verify concurrent safety" to our AI code review
checklist. Owner: @techlead. Due: Next Monday.

🎯 ACTION: Create a .prompt.md for our standard API test pattern.
Owner: @sarah. Due: End of week.

🎯 ACTION: Pair junior developers with seniors for their first
week of AI-assisted coding. Owner: @techlead. Due: Ongoing.

🎯 ACTION: Re-test the top 10 prompts after the model update.
Owner: @alex. Due: Next Wednesday.

🎯 ACTION: Set up monthly prompt showcase meeting.
Owner: @mike. Due: Book by Friday.
```

✅ **Good actions:** Specific, assigned, time-bound, achievable in one sprint
❌ **Bad actions:** "Use AI better" or "Be more careful with AI output" (not actionable)

---

## Retrospective Facilitation Guide

### Before the Retro (Facilitator Prep)

```markdown
## Pre-Retro Checklist

### 1 Week Before
- [ ] Send calendar invite (45-60 minutes)
- [ ] Share pre-retro survey (see template below)
- [ ] Review action items from last retro

### 1 Day Before
- [ ] Compile survey responses into themes
- [ ] Prepare the retro board (digital or physical)
- [ ] Review any AI-related incidents from the sprint
- [ ] Check: any new AI tools, updates, or policy changes?

### Day Of
- [ ] Ensure retro board is accessible to all participants
- [ ] Have a timer ready for timeboxed activities
- [ ] Prepare the "actions from last retro" review
```

### Pre-Retro Survey Template

Send this async survey 2-3 days before the retrospective:

```markdown
## AI Retro Pre-Survey (Sprint XX)

Please spend 5 minutes answering these questions before our retro:

1. **Biggest AI win this sprint?**
   [Free text]

2. **Biggest AI frustration this sprint?**
   [Free text]

3. **Did you discover any new prompts or techniques?**
   [Free text]

4. **Rate your AI productivity this sprint** (1-5)
   [ ] 1 — AI mostly got in the way
   [ ] 2 — Mixed results
   [ ] 3 — About the same as without AI
   [ ] 4 — Noticeably more productive
   [ ] 5 — Significantly more productive

5. **Any security or quality concerns?**
   [Free text]

6. **What topic should we discuss most in the retro?**
   [ ] Productivity patterns
   [ ] Quality/review process
   [ ] New tools/features
   [ ] Team practices/conventions
   [ ] Other: ___
```

### During the Retro

#### Agenda (45 minutes)

```
 0:00 - 0:05  Welcome + Review last retro's action items
 0:05 - 0:10  Share pre-survey summary and sprint AI stats
 0:10 - 0:25  Category brainstorm (Wins/Fails/Surprises/Learnings)
 0:25 - 0:35  Discussion + dot voting on themes
 0:35 - 0:45  Action items + assignment
```

#### Step-by-Step Facilitation

**1. Opening (5 min)**
- Review action items from the last retro
- Mark each as: ✅ Done | 🔄 In Progress | ❌ Not Started
- Celebrate completed actions

**2. Context Setting (5 min)**
- Share pre-survey summary (common themes)
- Share any relevant metrics (if available)
- Mention any AI tool changes since last retro

**3. Brainstorm Phase (15 min)**

Each participant adds sticky notes (physical or digital) to each category:

| 🏆 Wins | 💥 Fails | 😲 Surprises | 📚 Learnings |
|---------|---------|-------------|-------------|
| ... | ... | ... | ... |

Rules:
- One idea per note
- Be specific (not "AI was helpful" but "AI generated our migration scripts perfectly")
- No judging during brainstorm phase

**4. Discussion & Voting (10 min)**
- Group similar notes into themes
- Each person gets 3 dot votes
- Discuss the top-voted themes

**5. Action Items (10 min)**
- For each top theme, define a concrete action:
  - **What:** Specific change or experiment
  - **Who:** Owner (single person, not "the team")
  - **When:** Due date (within the next sprint)
- Limit to 3-5 actions (more = none get done)

### After the Retro

```markdown
## Post-Retro Checklist

- [ ] Publish retro notes to team wiki/docs within 24 hours
- [ ] Create tickets/issues for each action item
- [ ] Share key learnings in #ai-prompts Slack channel
- [ ] Update documentation if any conventions changed
- [ ] Add new prompts to the shared library if discovered
- [ ] Schedule follow-up for any complex actions
```

✅ **Good facilitation:** Structured, timeboxed, produces specific actions, follow-through on previous actions
❌ **Bad facilitation:** Open-ended discussion that runs long, produces vague commitments, ignores previous actions

---

## Digital Retro Board Template

### Miro / FigJam Board Layout

```
┌──────────────────────────────────────────────────────────────┐
│                    AI RETROSPECTIVE — Sprint XX               │
│                    Date: YYYY-MM-DD                           │
├──────────┬──────────┬──────────┬──────────┬──────────────────┤
│          │          │          │          │                   │
│ 🏆 WINS  │ 💥 FAILS │😲 SURPRISE│📚 LEARNING│   🎯 ACTIONS     │
│          │          │          │          │                   │
│ [sticky] │ [sticky] │ [sticky] │ [sticky] │  What:           │
│ [sticky] │ [sticky] │ [sticky] │ [sticky] │  Who:            │
│ [sticky] │ [sticky] │ [sticky] │ [sticky] │  When:           │
│ [sticky] │ [sticky] │          │ [sticky] │                  │
│          │          │          │          │  What:           │
│          │          │          │          │  Who:            │
│          │          │          │          │  When:           │
│          │          │          │          │                  │
├──────────┴──────────┴──────────┴──────────┼──────────────────┤
│           PREVIOUS ACTIONS REVIEW         │   METRICS        │
│                                           │                  │
│ ✅ Added security review prompt           │ AI Satisfaction:  │
│ 🔄 Pairing program for juniors           │ 3.8 / 5          │
│ ❌ Re-test prompt library (carry over)   │                  │
│                                           │ Prompts Shared:  │
│                                           │ 7 this sprint    │
└───────────────────────────────────────────┴──────────────────┘
```

### Board Setup Instructions

1. **Create the board** with the 5-column layout above
2. **Add a color legend:**
   - 🟢 Green stickies = positive (wins, good learnings)
   - 🔴 Red stickies = negative (fails, concerns)
   - 🟡 Yellow stickies = neutral (surprises, observations)
   - 🔵 Blue stickies = action items
3. **Pre-populate** with the pre-survey responses as conversation starters
4. **Set timer zones** in Miro for each phase of the retro
5. **Add voting dots** (3 per participant) for the prioritization phase

---

## Retrospective Frequency

### Recommended Cadence

| Type | Frequency | Duration | Focus |
|------|-----------|----------|-------|
| AI items in regular retro | Every sprint | 10 min (within regular retro) | Quick wins/fails, immediate actions |
| Dedicated AI retro | Monthly | 45-60 min | Deep dive on patterns and improvements |
| Quarterly AI review | Quarterly | 90 min | Strategic: tool decisions, ROI, team practices |
| Annual AI strategy | Yearly | Half day | Roadmap: tools, training, investment |

### Sprint-Level AI Check-In (10 minutes)

Add these questions to your regular sprint retro:

```markdown
## AI Quick Check (10 min)

1. 🏆 One AI win from this sprint? (2 min — round robin)
2. 💥 One AI frustration? (2 min — round robin)
3. 📚 One thing learned? (2 min — round robin)
4. 🎯 One action for next sprint? (4 min — team decision)
```

### Monthly Deep Dive (45-60 minutes)

Full AI retrospective using the facilitation guide above.

### Quarterly Strategy Review (90 minutes)

```markdown
## Quarterly AI Review Agenda

### Part 1: Retrospective on the Quarter (30 min)
- Review monthly retro themes and trends
- What improved? What didn't?
- Which action items had the biggest impact?

### Part 2: Metrics Review (15 min)
- AI productivity metrics (if tracking)
- Quality metrics (bugs, review findings)
- Adoption metrics (who's using what, how often)

### Part 3: Forward Planning (30 min)
- New AI tools or features to evaluate
- Training needs identified
- Process changes to propose
- Budget or license needs

### Part 4: Actions (15 min)
- Prioritize initiatives for next quarter
- Assign owners and milestones
- Agree on success criteria
```

✅ **Good cadence:** Regular lightweight check-ins plus periodic deep dives
❌ **Bad cadence:** Annual AI retrospective with no regular touchpoints (too infrequent to learn)

---

## Tracking Retrospective Outcomes

### Action Item Tracking

Maintain a running log of retro action items and their outcomes:

```markdown
## AI Retro Action Item Log

| Sprint | Action | Owner | Status | Outcome |
|--------|--------|-------|--------|---------|
| S23 | Add security review prompt | @sarah | ✅ Done | Caught 2 vulnerabilities in S24 |
| S23 | Pair juniors with seniors for AI | @lead | ✅ Done | Junior code quality improved |
| S23 | Re-test prompt library | @alex | 🔄 60% | 6 of 10 prompts re-tested |
| S24 | Create API test .prompt.md | @sarah | ✅ Done | Adopted by 5 team members |
| S24 | Add AI section to PR template | @mike | ❌ Dropped | Decided against (too much overhead) |
```

### Trend Analysis

Over time, look for patterns in your retro data:

**Positive Trends (things improving)**
- Fewer "AI generated buggy code" entries over time
- More sophisticated prompts being shared
- Less time spent on AI setup/configuration issues

**Concerning Trends (things to address)**
- Same complaints appearing sprint after sprint
- Action items consistently not completed
- Decreasing retro attendance or engagement
- Growing gap between heavy AI users and non-users

✅ **Good trend tracking:** "Race condition issues dropped 80% after adding concurrency review prompt in S23"
❌ **Bad trend tracking:** No tracking — each retro starts from scratch with no memory of past actions

---

## Common Retro Anti-Patterns

| Anti-Pattern | Symptom | Fix |
|-------------|---------|-----|
| **Groupthink** | Everyone agrees AI is great, no criticism | Explicitly ask for fails and frustrations |
| **Blame game** | "Person X doesn't review AI code carefully" | Focus on systems, not individuals |
| **Action overload** | 10+ actions items, none completed | Limit to 3-5 actions, enforce accountability |
| **Retro fatigue** | Low attendance, low engagement | Vary the format, shorten the meeting |
| **No follow-through** | Actions never reviewed, same issues repeat | Always start retro by reviewing last actions |
| **Too abstract** | "We should use AI better" | Require specific, observable changes |
| **All talk, no data** | Opinions without evidence | Track basic metrics, share data in retro |

---

## Retrospective Variations

### Format 1: Start/Stop/Continue

| Start Doing | Stop Doing | Continue Doing |
|-------------|-----------|----------------|
| Using .prompt.md files | Blindly accepting AI suggestions | Sharing wins in #ai-prompts |
| Monthly prompt showcase | Using AI for commit messages | Pairing for complex AI tasks |
| Tracking AI time savings | Over-engineering prompts | Security review for AI code |

### Format 2: Sailboat

```
      🏝️ ISLAND (Goal)               ⚓ ANCHOR (Holding us back)
   Higher AI productivity            Legacy code confuses AI
   Better code quality               Inconsistent instruction files
   Faster onboarding                 No shared prompt library

      💨 WIND (Helping us)            🪨 ROCKS (Risks ahead)
   Copilot getting better            Over-reliance on AI
   Team sharing more prompts          AI model changes break prompts
   Good instruction files             Junior devs not learning basics
```

### Format 3: Hot Air Balloon

- 🔥 **Hot air** (what's lifting us up): Effective AI practices
- 🧱 **Sandbags** (what's weighing us down): AI friction and failures
- ☀️ **Sunshine** (what makes us optimistic): New opportunities
- ⛈️ **Storm clouds** (what worries us): Risks and concerns

✅ **Good variation:** Switching formats keeps retros fresh and surfaces different insights
❌ **Bad variation:** Using the same format for a year until everyone is bored

---

## Getting Started Checklist

If you've never run an AI retrospective, start here:

### First AI Retrospective

- [ ] Schedule a 45-minute meeting with the team
- [ ] Send the pre-retro survey 3 days ahead
- [ ] Set up the retro board (digital or whiteboard with 5 columns)
- [ ] Prepare the "rules" slide (one idea per note, no judgment, be specific)
- [ ] Run through the facilitation guide above
- [ ] Capture 3 action items maximum
- [ ] Publish notes within 24 hours
- [ ] Schedule the next retro before ending this one

### Building the Habit

- [ ] Add a standing 10-minute "AI check" to sprint retros
- [ ] Schedule monthly deep-dive AI retros for the next quarter
- [ ] Assign a rotating facilitator (everyone takes a turn)
- [ ] Create a shared document for retro notes (append each session)
- [ ] Set up a Slack reminder for pre-retro survey distribution
- [ ] Review retro action items at weekly standup
