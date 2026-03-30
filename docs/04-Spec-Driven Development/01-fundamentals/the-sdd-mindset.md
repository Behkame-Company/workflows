# The SDD Mindset

> Thinking in specifications, not code — the mental shift required for spec-driven success.

---

## The Mindset Shift

Most developers think in code:

```
Traditional Developer Thinking:
  "I need a login page"
  → Opens editor
  → Starts writing components
  → Adds validation as they go
  → Tests when "it looks done"
```

SDD requires thinking in **specifications**:

```
SDD Developer Thinking:
  "I need a login page"
  → What are the exact requirements?
  → What constitutes success (acceptance criteria)?
  → What edge cases exist?
  → What are the constraints?
  → THEN plan the implementation
  → THEN write the code
```

---

## Five Mental Models for SDD

### 1. Blueprint Thinking
Think of specs as **architectural blueprints**. No builder starts constructing a house by laying random bricks. A blueprint defines:
- What rooms exist (features)
- Room dimensions (constraints)
- Plumbing and electrical routes (integrations)
- Building codes to comply with (non-functional requirements)

### 2. Contract Thinking
A spec is a **contract** between the product vision and the implementation:
- Clear deliverables
- Success criteria
- Boundaries (what's in scope, what's not)
- Remediation paths (error handling)

### 3. Test-Case Thinking
Before writing code, ask: **"How would I test this?"** If you can enumerate the test cases, you can write the spec. If you can't, the feature isn't well-defined yet.

### 4. User-Journey Thinking
Start with the user, not the technology:
```
Who is the user?
What do they want to accomplish?
What steps do they take?
What can go wrong at each step?
What does success look like?
```

### 5. Communication Thinking
A spec is **communication** — to your future self, to teammates, and to AI agents. Ask:
- Would a new team member understand this?
- Would an AI agent know exactly what to build?
- Would a QA engineer know what to test?

---

## Overcoming Resistance

### "Writing specs feels like busywork"
Reframe: You're not writing specs — you're **thinking through the problem**. The spec is a byproduct of thorough analysis. The 20 minutes you spend on a spec saves hours of rework.

### "I think better in code"
Prototype in code (vibe coding) to explore, then **formalize the insights** into a spec before building the production version. Use code to think, specs to communicate.

### "Requirements change too fast"
That's exactly why specs should be **lightweight and living**. A 20-line spec is easy to update. A scattered mental model distributed across Slack messages and code comments is not.

### "The AI understands me without specs"
It understands *something* — but not necessarily what you intended. Without specs, you can't verify whether the AI's interpretation matches your intent.

---

## The "Spec First" Habit

Build the habit with this daily practice:

```
Before starting any feature:
  □ Write one sentence: What am I building?
  □ Write three acceptance criteria
  □ Write two edge cases
  □ Write one constraint
  
Time investment: 5–10 minutes
Time saved: 30–60 minutes per feature
```

### The 5-Minute Spec

Even the simplest feature benefits from a 5-minute spec:

```markdown
## Feature: Password Reset

**What**: Users can reset their password via email link
**Success**: User receives email within 60s, link expires in 24h
**Edge cases**:
  - User requests multiple resets → only latest link works
  - Invalid/expired link → clear error message
**Constraints**: Rate limit to 3 requests per hour per email
```

This took 2 minutes to write and would prevent at least 3 bugs.

---

## Recognizing When You Need a Spec

| Signal | Action |
|--------|--------|
| "It's a simple feature" → 3 devs have 3 different ideas | Spec needed |
| PR review turns into a design discussion | Spec should have come first |
| AI generates code that "works" but misses the point | Spec context was insufficient |
| Stakeholder says "that's not what I meant" after demo | Acceptance criteria were missing |
| Feature shipped but nobody documented what it does | Living spec would have prevented this |

---

## SDD Maturity Model

| Level | Description | Practice |
|-------|-------------|----------|
| **0 — Ad Hoc** | No specs, pure vibe coding | "Let me just ask the AI..." |
| **1 — Reactive** | Specs written after incidents | "We should have specified that..." |
| **2 — Emerging** | Specs for complex features only | "This is big enough to spec out" |
| **3 — Practiced** | Specs for all production features | "Let me write the spec first" |
| **4 — Optimized** | Specs with quality gates and automation | "The spec drives the pipeline" |

Most teams that start SDD jump from Level 0 to Level 2 within a month.
