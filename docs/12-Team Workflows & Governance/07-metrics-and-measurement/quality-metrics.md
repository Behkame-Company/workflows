# Quality Metrics

> Tracking code quality impact when using AI-assisted development tools — from defect density and security findings to the subtle quality paradox where AI-generated code passes linting but harbors logic bugs.

---

## The Quality Question

AI tools can generate code that **looks** correct, **passes** linting, and **compiles** cleanly — but still contains subtle defects. Measuring quality in an AI-assisted world requires looking beyond surface-level indicators.

### The Quality Paradox

```
Traditional bad code:          AI-generated bad code:
─────────────────────          ─────────────────────
- Syntax errors                - Syntactically perfect
- Style violations             - Follows all style rules
- Missing semicolons           - Passes all linters
- Inconsistent formatting      - Well-formatted

Easy to catch ✅               Hard to catch ⚠️

But...                         But...
- Logic usually correct        - Subtle logic bugs
- Developer understands it     - Developer may not fully understand it
- Easy to debug                - Hard to debug (unfamiliar patterns)
- Fits the codebase            - May not fit architectural conventions
```

This is why quality metrics matter MORE, not less, with AI tools.

---

## Core Quality Metrics

### 1. Defect Density (Bugs per KLOC)

The most fundamental quality metric: how many bugs exist per thousand lines of code.

#### How to Measure

```
Defect Density = (Number of defects found) / (Lines of code / 1000)

Track separately:
- Defects found in code review
- Defects found in testing (QA/staging)
- Defects found in production
```

#### Tracking Template

```markdown
## Defect Density — [Period]

| Category | Before AI | After AI (Month 1-3) | After AI (Month 4-6) |
|----------|-----------|---------------------|---------------------|
| Review defects/KLOC | 4.2 | 5.1 | 3.8 |
| Testing defects/KLOC | 1.8 | 2.3 | 1.5 |
| Production defects/KLOC | 0.4 | 0.6 | 0.3 |
| **Total defects/KLOC** | **6.4** | **8.0** | **5.6** |

### Interpretation
Month 1-3: Defect density INCREASED 25% ⚠️
  - Expected during AI adoption learning curve
  - Developers accepting AI code without sufficient review
  - Action: Reinforce review practices for AI-generated code

Month 4-6: Defect density DECREASED 12% below baseline ✅
  - Team learned to review AI output effectively
  - AI generates better test coverage, catching bugs earlier
  - AI-assisted code review catches patterns humans miss
```

#### ✅ Good: Track Defect Types, Not Just Counts

```markdown
## Defect Classification — Month 6 (AI-Assisted)

| Defect Type | Count | % | Before AI % | Change |
|------------|-------|---|-------------|--------|
| Logic errors | 12 | 35% | 28% | ▲ More logic bugs |
| Boundary conditions | 8 | 24% | 12% | ▲ AI misses edge cases |
| Integration issues | 5 | 15% | 18% | ▼ Fewer integration bugs |
| Null/undefined handling | 4 | 12% | 22% | ▼ AI handles null checks |
| Concurrency | 2 | 6% | 8% | ▼ Slight improvement |
| Other | 3 | 9% | 12% | — |

### Insight
AI REDUCES: null handling bugs, integration issues
AI INCREASES: logic errors, boundary condition misses

Action: Add boundary condition testing to AI review checklist
```

#### ❌ Bad: Only Track Total Defect Count

```
"We had 24 bugs this month, 22 last month. Quality is fine."

(Doesn't account for: code volume shipped, severity of bugs,
WHERE bugs were caught, or TYPE of bugs.)
```

---

### 2. Security Findings (SAST Results Trend)

Static Application Security Testing results over time. Critical for AI-generated code, which can introduce known vulnerability patterns.

#### What to Track

| Metric | How to Measure | Warning Threshold |
|--------|---------------|------------------|
| Critical findings | SAST scan count | Any increase |
| High findings | SAST scan count | >10% increase |
| New finding types | Classification analysis | New categories appearing |
| False positive rate | Manual review | >30% false positives |
| Time to remediate | Finding age tracking | >5 days for critical |

#### Security Findings Dashboard

```markdown
## Security Scan Trends — [Quarter]

### SAST Findings by Severity
| Severity | Q-2 (Baseline) | Q-1 | Current | Trend |
|----------|----------------|-----|---------|-------|
| Critical | 0 | 0 | 1 | ⚠️ Investigate |
| High | 3 | 5 | 4 | ► Stable |
| Medium | 12 | 15 | 11 | ✅ Improving |
| Low | 28 | 32 | 25 | ✅ Improving |

### AI-Specific Security Concerns
| Issue | Occurrences | Action |
|-------|-------------|--------|
| Hardcoded secrets in AI code | 2 | Reinforce: never include secrets in prompts |
| SQL injection patterns | 1 | AI used string concatenation instead of parameterized queries |
| Insecure deserialization | 0 | ✅ No issues |
| Path traversal | 1 | AI didn't validate user-provided file paths |
| XSS patterns | 0 | ✅ No issues |
```

#### ✅ Good: Track AI-Specific Vulnerability Patterns

```
Common AI security mistakes to watch for:
1. Using string interpolation in SQL queries
2. Not validating/sanitizing user input
3. Hardcoding API keys or example credentials
4. Using deprecated/vulnerable library versions
5. Missing authentication/authorization checks
6. Insecure default configurations

Add these to your SAST rules and review checklists.
```

#### ❌ Bad: Assume AI Code Is Secure Because It Compiles

```
"Copilot generated the authentication module and all tests pass."

(Nobody checked whether the JWT implementation uses a strong
algorithm, whether tokens expire appropriately, or whether
the refresh token rotation is actually secure.)
```

---

### 3. Test Coverage

Does AI help or hurt test coverage? AI tools can generate tests quickly, but the quality of those tests matters more than the quantity.

#### What to Track

```markdown
## Test Coverage Analysis — [Period]

### Overall Coverage
| Metric | Baseline | Current | Target | Status |
|--------|----------|---------|--------|--------|
| Line coverage | 72% | 81% | 80% | ✅ Met |
| Branch coverage | 58% | 69% | 70% | ⚠️ Close |
| Function coverage | 85% | 91% | 90% | ✅ Met |

### Coverage by Code Origin
| Code Origin | Line Coverage | Branch Coverage | Note |
|-------------|--------------|-----------------|------|
| Human-written code | 76% | 64% | Higher intent coverage |
| AI-assisted code | 84% | 72% | Higher raw coverage |
| AI-generated tests | N/A | N/A | Covering 68% of codebase |

### Test Quality Assessment (Monthly Sampling)
| Quality Metric | Score | Target | Status |
|---------------|-------|--------|--------|
| Mutation testing survival | 34% | <40% | ✅ Good |
| Tests with meaningful assertions | 89% | >85% | ✅ Good |
| Tests that test implementation (not behavior) | 18% | <15% | ⚠️ Some brittle tests |
| Flaky tests introduced | 3 | 0 | ❌ Investigate |
```

#### ✅ Good: Measure Test Quality, Not Just Coverage

```python
# Mutation testing reveals whether tests actually catch bugs
# Run periodically (weekly or per sprint)

# Example: pytest-mutmut configuration
# mutmut.toml
[tool.mutmut]
paths_to_mutate = "src/"
tests_dir = "tests/"
runner = "python -m pytest"

# A high mutation survival rate means tests are superficial
# Target: <40% of mutants survive (i.e., >60% of mutations are caught)
```

#### ❌ Bad: Celebrate Coverage Numbers Alone

```
"AI helped us go from 72% to 85% coverage! Quality is great!"

(Many AI-generated tests assert that functions "don't throw"
or check return type rather than testing actual behavior.
They inflate coverage without testing meaningful scenarios.)
```

---

### 4. Code Review Findings

Track what human reviewers find during code review. The types of issues found — not just the count — reveal how AI affects code quality.

#### Review Findings Classification

```markdown
## Review Findings Analysis — [Month]

### Findings by Category
| Category | AI-Assisted PRs | Non-AI PRs | Interpretation |
|----------|----------------|------------|----------------|
| Style/formatting | 0.3/PR | 1.2/PR | AI handles style ✅ |
| Naming issues | 0.5/PR | 0.8/PR | AI slightly better ✅ |
| Logic errors | 1.4/PR | 0.7/PR | AI introduces more ⚠️ |
| Missing error handling | 1.1/PR | 0.6/PR | AI skips edge cases ⚠️ |
| Security concerns | 0.3/PR | 0.2/PR | Slightly more ⚠️ |
| Architecture/design | 0.8/PR | 0.9/PR | Similar |
| Missing tests | 0.2/PR | 0.7/PR | AI generates tests ✅ |
| Documentation gaps | 0.1/PR | 0.5/PR | AI documents well ✅ |
| **Total** | **4.7/PR** | **5.6/PR** | **16% fewer total** |

### Key Insight
AI-assisted PRs have FEWER total findings but MORE findings in:
- Logic errors (2x more frequent)
- Missing error handling (1.8x more frequent)

Action: Update review checklist to emphasize logic verification
and error handling for AI-assisted PRs.
```

#### ✅ Good: Track the Shift in Reviewer Focus

```
Before AI: Reviewers spent ~40% of time on style and formatting.
After AI:  Reviewers spend ~10% on style, ~50% on logic and design.

This is a POSITIVE shift. Humans focus on what humans do best.
```

#### ❌ Bad: Track Only "Number of Review Comments"

```
"Review comments per PR dropped from 5.6 to 4.7. AI is helping."

(Or reviewers are doing less thorough reviews of AI code
because it "looks right" and they assume AI is correct.)
```

---

### 5. Technical Debt (SonarQube Trends)

Is AI-generated code adding to or reducing your technical debt? Track through static analysis tools.

#### Technical Debt Dashboard

```markdown
## Technical Debt Tracking — [Quarter]

### SonarQube Quality Gate
| Metric | Baseline | Current | Gate | Status |
|--------|----------|---------|------|--------|
| Reliability | B | A | A | ✅ Improved |
| Security | A | A | A | ✅ Maintained |
| Maintainability | B | B | A | ⚠️ Not improved |
| Coverage | 72% | 81% | 80% | ✅ Met |
| Duplications | 4.2% | 6.8% | 5% | ❌ Increased |

### Debt Trend (in developer-days)
| Category | Baseline | Current | Change |
|----------|----------|---------|--------|
| Bug debt | 8 days | 5 days | ▼ Improved |
| Vulnerability debt | 2 days | 2 days | ► Stable |
| Code smell debt | 15 days | 22 days | ▲ Increased ⚠️ |
| **Total** | **25 days** | **29 days** | **▲ 16%** |

### AI-Related Debt Patterns
| Pattern | Occurrences | Impact |
|---------|-------------|--------|
| Code duplication (AI generates similar code in multiple places) | 14 | High |
| Overly complex functions (AI writes long functions) | 8 | Medium |
| Unused imports/variables | 11 | Low |
| Inconsistent error handling | 6 | Medium |
```

#### ✅ Good: Watch for AI-Specific Debt Patterns

```
AI tends to introduce specific types of technical debt:

1. DUPLICATION: AI generates similar solutions independently
   for similar problems, creating copy-paste-like patterns.
   → Track code duplication percentage closely.

2. OVER-ENGINEERING: AI sometimes generates unnecessarily
   complex solutions for simple problems.
   → Watch function complexity metrics (cyclomatic complexity).

3. INCONSISTENCY: AI may use different patterns for similar
   tasks across the codebase.
   → Track style consistency metrics.

4. DEAD CODE: AI generates complete implementations that may
   include unused branches or functions.
   → Track dead code metrics.
```

#### ❌ Bad: Ignore Debt Because "AI Code Passes All Checks"

```
"SonarQube shows no critical issues. AI code quality is fine."

(Meanwhile, code duplication is up 60%, complexity is climbing,
and three developers have complained about unreadable AI code.)
```

---

### 6. Production Incidents

The ultimate quality metric: what breaks in production?

#### Production Incident Tracking

```markdown
## Production Incident Analysis — [Quarter]

### Incident Summary
| Severity | Baseline Q | Current Q | Change |
|----------|-----------|-----------|--------|
| P1 (Critical) | 1 | 0 | ✅ Improved |
| P2 (Major) | 3 | 4 | ⚠️ Slight increase |
| P3 (Minor) | 8 | 7 | ► Stable |
| **Total** | **12** | **11** | **► Stable** |

### AI-Related Incident Analysis
| Incident | Root Cause | AI Involvement | Lesson |
|----------|-----------|---------------|--------|
| API timeout | Missing pagination in AI-generated query | Direct | AI didn't consider data volume at scale |
| Data corruption | Race condition in concurrent handler | Direct | AI-generated code wasn't thread-safe |
| Feature flag leak | Incorrect condition logic | Indirect | AI suggestion accepted without review |
| Auth bypass | — | None | Pre-existing issue |

### Mean Time to Recovery (MTTR)
| Metric | Baseline | Current | Change |
|--------|----------|---------|--------|
| MTTR (all incidents) | 4.2 hrs | 3.8 hrs | ▼ Improved |
| MTTR (AI-related) | N/A | 5.1 hrs | Higher than avg ⚠️ |

### Insight
AI-related incidents take LONGER to resolve because:
- Code is less familiar to the developer debugging it
- AI patterns may not match team's mental models
- Root cause is often subtle (logic, not syntax)
```

#### ✅ Good: Track AI Involvement in Incidents

```
For every production incident, ask:
1. Was AI-generated code involved? (Direct / Indirect / None)
2. Would a human have made the same mistake?
3. Could better AI review practices have caught this?
4. Was the root cause a known AI weakness (e.g., edge cases)?

This data improves your review checklists and AI usage guidelines.
```

#### ❌ Bad: Blame AI for All Issues (or Never Blame AI)

```
Bad: "AI caused this outage! We should stop using AI tools!"
(The actual cause was a configuration error unrelated to AI.)

Also bad: "AI code never causes production issues."
(Nobody is tracking whether incidents involve AI-generated code.)
```

---

## Focus Areas: Where AI Quality Issues Hide

### Boundary Conditions

AI frequently misses edge cases and boundary conditions. Actively test for these.

```python
# AI might generate this:
def calculate_discount(price, quantity):
    if quantity > 10:
        return price * 0.9  # 10% discount
    return price

# Missing boundary conditions:
# - What if price is negative?
# - What if quantity is 0?
# - What if quantity is exactly 10? (off-by-one)
# - What if price is None?
# - What about floating point precision?

# Better version (what review should catch):
def calculate_discount(price: float, quantity: int) -> float:
    if price < 0:
        raise ValueError("Price cannot be negative")
    if quantity < 0:
        raise ValueError("Quantity cannot be negative")
    if quantity > 10:
        return round(price * 0.9, 2)
    return round(price, 2)
```

### Error Handling Completeness

AI often generates "happy path" code and skips error handling.

```typescript
// AI might generate this:
async function fetchUserData(userId: string) {
  const response = await fetch(`/api/users/${userId}`);
  const data = await response.json();
  return data;
}

// Missing error handling:
// - What if the network request fails?
// - What if the response is not JSON?
// - What if the response is 404 or 500?
// - What if userId is empty/null?

// Better version:
async function fetchUserData(userId: string): Promise<UserData> {
  if (!userId?.trim()) {
    throw new Error("userId is required");
  }

  const response = await fetch(`/api/users/${encodeURIComponent(userId)}`);

  if (!response.ok) {
    throw new Error(
      `Failed to fetch user ${userId}: ${response.status} ${response.statusText}`
    );
  }

  try {
    return await response.json() as UserData;
  } catch {
    throw new Error(`Invalid JSON response for user ${userId}`);
  }
}
```

### Integration Correctness

AI generates code that works in isolation but may not integrate correctly with the rest of the system.

```
Common integration issues in AI-generated code:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Database transactions — AI may not wrap related operations
2. Event ordering — AI may not preserve required event sequences
3. Cache invalidation — AI may update DB but not invalidate cache
4. API contracts — AI may drift from the expected API shape
5. Authentication context — AI may not propagate auth tokens
6. Logging context — AI may break correlation IDs or trace spans
```

---

## Quality Dashboard Template

```
┌───────────────────────────────────────────────────────┐
│           CODE QUALITY DASHBOARD — [Team]             │
├───────────────────────┬───────────────────────────────┤
│  DEFECT DENSITY       │  SECURITY                     │
│  ─────────────        │  ────────                     │
│  Review: 3.8/KLOC     │  Critical: 0 ✅               │
│  (baseline: 4.2)      │  High: 4 ►                    │
│  ▼ 10% ✅             │  Medium: 11 ✅                 │
│                       │  (baseline: 12)               │
│  Prod: 0.3/KLOC       │                               │
│  (baseline: 0.4)      │  Remediation: 3.2 days        │
│  ▼ 25% ✅             │  (target: <5 days)            │
├───────────────────────┼───────────────────────────────┤
│  TEST COVERAGE        │  TECHNICAL DEBT               │
│  ─────────────        │  ──────────────               │
│  Line: 81% ✅         │  Total: 29 dev-days           │
│  Branch: 69% ⚠️       │  (baseline: 25 days)          │
│  Mutation: 66% ✅      │  ▲ 16% ⚠️                    │
│                       │                               │
│  Flaky tests: 3 ❌    │  Duplication: 6.8% ❌          │
│  (target: 0)          │  (target: <5%)                │
├───────────────────────┼───────────────────────────────┤
│  PRODUCTION           │  REVIEW QUALITY               │
│  ──────────           │  ──────────────               │
│  P1 incidents: 0 ✅   │  Logic findings: 1.4/PR ⚠️    │
│  P2 incidents: 4 ⚠️   │  Error handling: 1.1/PR ⚠️    │
│  MTTR: 3.8 hrs        │  Style issues: 0.3/PR ✅      │
│  AI-related: 2 of 4   │  Missing tests: 0.2/PR ✅     │
├───────────────────────┴───────────────────────────────┤
│  THRESHOLDS                                           │
│  ──────────                                           │
│  🟢 Green: Metric at or better than target            │
│  🟡 Yellow: Within 10% of threshold                   │
│  🔴 Red: Exceeds threshold — action required          │
└───────────────────────────────────────────────────────┘
```

### Quality Thresholds Configuration

```yaml
# quality-thresholds.yml
# Define alert thresholds for quality metrics

defect_density:
  review_per_kloc:
    green: "<= 4.0"
    yellow: "<= 6.0"
    red: "> 6.0"
  production_per_kloc:
    green: "<= 0.3"
    yellow: "<= 0.6"
    red: "> 0.6"

security:
  critical_findings:
    green: "0"
    yellow: "N/A"
    red: ">= 1"
  high_findings:
    green: "<= 3"
    yellow: "<= 6"
    red: "> 6"
  remediation_days:
    green: "<= 3"
    yellow: "<= 5"
    red: "> 5"

test_coverage:
  line_coverage:
    green: ">= 80%"
    yellow: ">= 70%"
    red: "< 70%"
  branch_coverage:
    green: ">= 70%"
    yellow: ">= 60%"
    red: "< 60%"
  mutation_survival:
    green: "<= 35%"
    yellow: "<= 45%"
    red: "> 45%"

technical_debt:
  total_days:
    green: "<= baseline"
    yellow: "<= baseline * 1.2"
    red: "> baseline * 1.2"
  duplication:
    green: "<= 5%"
    yellow: "<= 8%"
    red: "> 8%"

production:
  p1_incidents:
    green: "0"
    yellow: "1"
    red: ">= 2"
  mttr_hours:
    green: "<= 4"
    yellow: "<= 8"
    red: "> 8"
```

---

## GitHub Actions: Quality Gate Workflow

```yaml
# .github/workflows/quality-metrics.yml
name: Quality Metrics Collection

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'  # Weekly Monday 6 AM UTC

jobs:
  quality-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Tests with Coverage
        run: |
          npm ci
          npm run test -- --coverage --coverageReporters=json-summary

      - name: Check Coverage Thresholds
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | \
            python3 -c "import sys,json; \
            d=json.load(sys.stdin)['total']; \
            print(f\"Lines: {d['lines']['pct']}% | \
            Branches: {d['branches']['pct']}%\")")
          echo "$COVERAGE"

      - name: Count TODO/FIXME/HACK
        run: |
          echo "## Code Health Indicators"
          echo "TODOs: $(grep -r 'TODO' src/ --include='*.ts' | wc -l)"
          echo "FIXMEs: $(grep -r 'FIXME' src/ --include='*.ts' | wc -l)"
          echo "HACKs: $(grep -r 'HACK' src/ --include='*.ts' | wc -l)"

      - name: Check for Code Duplication
        run: |
          npx jscpd src/ --reporters json --output .
          cat jscpd-report.json | python3 -c "
          import sys, json
          data = json.load(sys.stdin)
          stats = data.get('statistics', {})
          total = stats.get('total', {})
          dup_pct = total.get('duplicatedLines', 0) / max(total.get('lines', 1), 1) * 100
          print(f'Duplication: {dup_pct:.1f}%')
          if dup_pct > 5:
              print('::warning::Code duplication exceeds 5% threshold')
          "

      - name: Generate Quality Report
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const coverageData = JSON.parse(
              fs.readFileSync('coverage/coverage-summary.json', 'utf8')
            );
            const total = coverageData.total;

            const report = {
              date: new Date().toISOString().split('T')[0],
              coverage: {
                lines: total.lines.pct,
                branches: total.branches.pct,
                functions: total.functions.pct,
                statements: total.statements.pct,
              },
            };

            fs.writeFileSync(
              'quality-report.json',
              JSON.stringify(report, null, 2)
            );

      - name: Upload Quality Report
        uses: actions/upload-artifact@v4
        with:
          name: quality-metrics-${{ github.run_id }}
          path: quality-report.json
          retention-days: 90
```

---

## Quality Improvement Actions

### When Defect Density Increases

```markdown
## Defect Density Action Plan

1. **Identify the source**
   - Are defects in AI-generated code or human code?
   - Which types of defects increased?
   - Which task types or features are affected?

2. **Reinforce review practices**
   - [ ] Update review checklist with common AI defect types
   - [ ] Add "boundary condition check" to review template
   - [ ] Require reviewers to verify error handling completeness

3. **Improve AI usage**
   - [ ] Share examples of AI code that had defects
   - [ ] Add prompt templates that emphasize edge cases
   - [ ] Encourage "ask AI to find bugs in its own code"

4. **Add automated guards**
   - [ ] Add mutation testing to CI pipeline
   - [ ] Add property-based testing for critical functions
   - [ ] Enhance SAST rules for common AI patterns
```

### When Test Quality Declines

```markdown
## Test Quality Action Plan

1. **Audit AI-generated tests**
   - [ ] Sample 20 AI-generated tests — are assertions meaningful?
   - [ ] Run mutation testing — what percentage of mutants survive?
   - [ ] Check for flaky tests — are AI tests deterministic?

2. **Set test quality standards**
   - [ ] Each test must have at least one behavioral assertion
   - [ ] No "test that it doesn't throw" as the only assertion
   - [ ] Test names must describe the BEHAVIOR being verified

3. **Train the team**
   - [ ] Show examples of good vs bad AI-generated tests
   - [ ] Share prompt templates that produce better tests
   - [ ] Practice: "Given AI tests, what's missing?"
```

---

## Common Quality Measurement Mistakes

### ✅ Do: Measure What AI Doesn't Catch

```
Run experiments: Deliberately introduce bugs and see what catches them.

AI-assisted review vs human-only review:
- Which catches more logic errors?
- Which catches more edge cases?
- Which catches more security issues?

This tells you where to focus human review effort.
```

### ❌ Don't: Equate "Passes Linting" with "High Quality"

```
AI-generated code that passes ESLint, Prettier, and TypeScript
compilation can still have:
- Incorrect business logic
- Race conditions
- Memory leaks
- Security vulnerabilities
- Unhandled error states
- Broken integration with other components
```

### ✅ Do: Track Quality Over Time, Not Just Snapshots

```
A single month's quality data means nothing.
Track trends over 3-6 months to see real patterns.
Short-term fluctuations are normal.
```

### ❌ Don't: Set Unrealistic Quality Targets

```
"Zero defects" is unrealistic and counterproductive.
Instead, set targets relative to your baseline:
- Defect density should not exceed baseline by >10%
- Production incidents should not increase
- Security findings should trend downward
```

### ✅ Do: Separate AI Quality from Overall Quality

```
Track metrics for AI-assisted PRs vs non-AI PRs separately.
This isolates AI's effect from other variables.
```

### ❌ Don't: Forget Qualitative Quality Signals

```
Numbers don't capture everything. Also pay attention to:
- "I can't understand this AI-generated code" (maintainability)
- "This works but feels wrong" (code smell)
- "AI used a completely different pattern than the rest of the codebase"
- "The tests pass but I don't believe they test the right things"
```

---

## Quality Metrics Checklist

- [ ] **Defect tracking** — Classify defects by type and AI involvement
- [ ] **Security scanning** — SAST/DAST running on every PR with trend tracking
- [ ] **Test coverage** — Line, branch, and mutation testing tracked
- [ ] **Review findings** — Categorize review comments by type
- [ ] **Technical debt** — SonarQube or equivalent running continuously
- [ ] **Production incidents** — Track AI involvement in every incident
- [ ] **Quality thresholds** — Defined and enforced in CI/CD
- [ ] **Monthly review** — Analyze quality trends and take action
- [ ] **Team awareness** — Share quality data and improvement actions
- [ ] **Continuous improvement** — Update review checklists based on findings
