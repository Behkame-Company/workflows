# SDD vs TDD vs BDD

> How Spec-Driven Development relates to test-first methodologies — and how to combine them.

---

## Quick Definitions

| Methodology | Write First | Focus | Key Artifact |
|-------------|------------|-------|--------------|
| **SDD** | Specification | What to build, why, for whom | `spec.md` |
| **TDD** | Unit tests | Code correctness | Test files |
| **BDD** | Behavior scenarios | Business outcomes | `.feature` files |

---

## The Lifecycle Position

Each methodology operates at a different point in the development lifecycle:

```
                 SDD                    BDD              TDD
                  │                      │                │
          What to build            How it behaves    How it works
                  │                      │                │
    ┌─────────────▼──────────────────────▼────────────────▼──────────┐
    │  Spec.md → Plan.md → Tasks.md → Features → Unit Tests → Code  │
    └───────────────────────────────────────────────────────────────────┘
                               ◄────── Implementation ──────►
```

**SDD operates before code exists.** TDD and BDD operate during implementation.

---

## Detailed Comparison

| Dimension | SDD | TDD | BDD |
|-----------|-----|-----|-----|
| **Operates at** | Requirements level | Code level | Behavior level |
| **Written by** | Product + Engineering | Engineers | Product + QA + Engineering |
| **Language** | Natural language + structured sections | Programming language | Gherkin (Given/When/Then) |
| **Validates** | What to build | That code works correctly | That features meet business needs |
| **When written** | Before any code | Before each function/module | Before each feature |
| **AI synergy** | Excellent — rich context for agents | Good — tests guide implementation | Good — scenarios define expectations |
| **Maintenance** | Must update specs when requirements change | Must update tests when code changes | Must update scenarios when behavior changes |

---

## How They Complement Each Other

SDD, TDD, and BDD are **not competing approaches** — they form layers:

```
┌─────────────────────────────────────────┐
│  SDD: Specification Layer               │
│  "Build a user registration system      │
│   with email verification, rate          │
│   limiting, and GDPR compliance"         │
├─────────────────────────────────────────┤
│  BDD: Behavior Layer                    │
│  Given a user submits a valid email      │
│  When the registration form is submitted │
│  Then a verification email is sent       │
│  And the account is created as pending   │
├─────────────────────────────────────────┤
│  TDD: Implementation Layer              │
│  test("createUser stores pending user")  │
│  test("sendVerificationEmail sends")     │
│  test("rateLimiter blocks after 5/min")  │
└─────────────────────────────────────────┘
```

### The Combined Workflow

1. **SDD**: Write spec with requirements, acceptance criteria, edge cases
2. **BDD**: Derive behavior scenarios from acceptance criteria
3. **TDD**: Implement each scenario test-first at the code level
4. **Validate**: Check implementation against all three layers

---

## SDD Generates BDD Scenarios

One powerful pattern: **use specs to auto-generate BDD scenarios**.

### From Spec:
```markdown
## Acceptance Criteria
- Users can register with email and password
- Passwords must be 8+ characters with 1 number
- Duplicate emails show "already registered" error
- Verification email sent within 30 seconds
```

### Generated BDD:
```gherkin
Feature: User Registration

  Scenario: Successful registration
    Given I am on the registration page
    When I enter "user@example.com" and "Passw0rd!"
    Then I should see "Check your email"
    And a verification email should be sent within 30 seconds

  Scenario: Weak password
    Given I am on the registration page
    When I enter "user@example.com" and "weak"
    Then I should see "Password must be 8+ characters with 1 number"

  Scenario: Duplicate email
    Given "user@example.com" is already registered
    When I enter "user@example.com" and "Passw0rd!"
    Then I should see "already registered"
```

---

## When to Use Each

| Situation | Recommended Approach |
|-----------|---------------------|
| Starting a new product | SDD → BDD → TDD |
| Adding a complex feature | SDD → TDD (skip BDD if team is small) |
| Fixing a bug | TDD only (write failing test → fix → verify) |
| Compliance/audit project | SDD + BDD (full traceability) |
| Internal tooling | TDD only (requirements are obvious) |
| API design | SDD + TDD (spec defines contract) |
| UI feature | SDD + BDD (behavior matters most) |

---

## Anti-Pattern: Choosing Only One

> ❌ "We do TDD so we don't need specs"

TDD tests what the developer *anticipated*. Without specs, you miss:
- Cross-functional requirements
- Edge cases only visible at the product level
- Non-functional requirements (performance, security, accessibility)
- Stakeholder alignment on what "done" means

> ❌ "We write specs so we don't need tests"

Specs validate *intent*. Tests validate *behavior*. You need both.

---

## The AI-Era Insight

With AI coding agents, the combination becomes even more powerful:

```
SDD:  Spec provides rich context → AI generates better code
BDD:  Scenarios provide validation criteria → AI generates testable code
TDD:  Tests provide immediate feedback → AI can self-correct
```

**All three together** create a self-reinforcing loop where the AI has maximum context, clear success criteria, and automated validation.

---

*Next: [Core Principles →](core-principles.md)*
