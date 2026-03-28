# Productivity Metrics

> Practical approaches to measuring development speed and throughput when using AI-assisted coding tools — from DORA metrics to AI-specific indicators, with automation scripts and dashboard configurations.

---

## Why Productivity Metrics Matter

"Are we going faster?" is the first question leadership asks about AI tools. It sounds simple but measuring developer productivity is notoriously difficult. AI tools add another layer of complexity: they can make individual tasks faster while potentially introducing hidden costs.

This guide covers what to measure, how to measure it, and — critically — what NOT to measure.

---

## Core Productivity Metrics

### 1. Cycle Time (Idea to Production)

Cycle time is the gold standard for delivery speed. It measures the full journey from work item creation to production deployment.

#### What It Measures

```
Cycle Time = Time from "work starts" to "code in production"

Breakdown:
┌──────────┬───────────┬───────────┬───────────┬──────────┐
│  Coding  │  Review   │   CI/CD   │  Staging  │  Deploy  │
│  Time    │  Time     │   Time    │  Testing  │  Time    │
└──────────┴───────────┴───────────┴───────────┴──────────┘
     ▲           ▲           ▲           ▲           ▲
     │           │           │           │           │
  AI helps    AI helps    Unchanged   Unchanged   Unchanged
  here most   here too
```

#### ✅ Good: Track Cycle Time Components Separately

```markdown
## Cycle Time Breakdown — Sprint 14

| Phase | Before AI | After AI | Change |
|-------|-----------|----------|--------|
| Coding | 2.8 days | 1.9 days | ▼ 32% ✅ |
| Review | 1.1 days | 0.8 days | ▼ 27% ✅ |
| CI/CD | 0.3 days | 0.3 days | — |
| Staging | 0.5 days | 0.5 days | — |
| Deploy | 0.2 days | 0.2 days | — |
| **Total** | **4.9 days** | **3.7 days** | **▼ 24%** |

Insight: AI primarily reduces coding and review time.
Build, staging, and deploy are infrastructure-bound.
```

#### ❌ Bad: Track Only Total Cycle Time

```
Cycle time went from 4.9 days to 3.7 days. AI is working!

(But you also parallelized CI and added auto-deploy last month,
so the improvement might not be from AI at all.)
```

#### Automation: Collecting Cycle Time Data

```python
#!/usr/bin/env python3
"""
collect_cycle_time.py
Collects PR cycle time data from GitHub API for AI impact analysis.

Usage: python collect_cycle_time.py --repo owner/repo --weeks 4
"""

import argparse
import json
import os
from datetime import datetime, timedelta, timezone

# Requires: pip install requests
import requests

GITHUB_TOKEN = os.environ.get("GITHUB_TOKEN")
HEADERS = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github.v3+json"
}

def get_merged_prs(owner, repo, since_date):
    """Fetch all PRs merged since the given date."""
    prs = []
    page = 1
    while True:
        url = f"https://api.github.com/repos/{owner}/{repo}/pulls"
        params = {
            "state": "closed",
            "sort": "updated",
            "direction": "desc",
            "per_page": 100,
            "page": page
        }
        response = requests.get(url, headers=HEADERS, params=params)
        response.raise_for_status()
        data = response.json()

        if not data:
            break

        for pr in data:
            if pr["merged_at"]:
                merged_date = datetime.fromisoformat(
                    pr["merged_at"].replace("Z", "+00:00")
                )
                if merged_date >= since_date:
                    prs.append(pr)
                elif merged_date < since_date:
                    return prs

        page += 1

    return prs


def calculate_cycle_time(pr):
    """Calculate cycle time components for a single PR."""
    created = datetime.fromisoformat(
        pr["created_at"].replace("Z", "+00:00")
    )
    merged = datetime.fromisoformat(
        pr["merged_at"].replace("Z", "+00:00")
    )

    total_hours = (merged - created).total_seconds() / 3600

    return {
        "pr_number": pr["number"],
        "title": pr["title"],
        "author": pr["user"]["login"],
        "created_at": pr["created_at"],
        "merged_at": pr["merged_at"],
        "total_hours": round(total_hours, 1),
        "total_days": round(total_hours / 24, 1),
        "additions": pr.get("additions", 0),
        "deletions": pr.get("deletions", 0),
    }


def generate_report(cycle_times):
    """Generate summary statistics from cycle time data."""
    if not cycle_times:
        return {"error": "No PRs found in the specified period"}

    total_days = [ct["total_days"] for ct in cycle_times]

    return {
        "period_prs": len(cycle_times),
        "avg_cycle_days": round(sum(total_days) / len(total_days), 1),
        "median_cycle_days": round(
            sorted(total_days)[len(total_days) // 2], 1
        ),
        "min_cycle_days": min(total_days),
        "max_cycle_days": max(total_days),
        "p90_cycle_days": round(
            sorted(total_days)[int(len(total_days) * 0.9)], 1
        ),
    }


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Collect PR cycle time metrics"
    )
    parser.add_argument("--repo", required=True, help="owner/repo")
    parser.add_argument(
        "--weeks", type=int, default=4, help="Weeks to look back"
    )
    args = parser.parse_args()

    owner, repo = args.repo.split("/")
    since = datetime.now(timezone.utc) - timedelta(weeks=args.weeks)

    print(f"Collecting PRs merged since {since.date()} for {args.repo}...")
    prs = get_merged_prs(owner, repo, since)
    cycle_times = [calculate_cycle_time(pr) for pr in prs]
    report = generate_report(cycle_times)

    output = {
        "repository": args.repo,
        "period_weeks": args.weeks,
        "since": since.isoformat(),
        "summary": report,
        "details": cycle_times,
    }

    output_file = f"cycle-time-report-{datetime.now().strftime('%Y%m%d')}.json"
    with open(output_file, "w") as f:
        json.dump(output, f, indent=2)

    print(f"\nSummary:")
    print(f"  PRs merged: {report.get('period_prs', 0)}")
    print(f"  Avg cycle time: {report.get('avg_cycle_days', 'N/A')} days")
    print(f"  Median cycle time: {report.get('median_cycle_days', 'N/A')} days")
    print(f"  P90 cycle time: {report.get('p90_cycle_days', 'N/A')} days")
    print(f"\nFull report saved to {output_file}")
```

---

### 2. PR Throughput (PRs Merged Per Sprint)

Measures how many pull requests the team merges per time period. Simple to collect, easy to track over time.

#### How to Interpret

| Scenario | What It Means |
|----------|--------------|
| PRs up, quality stable | ✅ AI is helping ship faster |
| PRs up, quality down | ⚠️ AI is generating more but lower quality code |
| PRs up, PR size down | ✅ AI enables smaller, more frequent PRs (good!) |
| PRs unchanged | AI may not be impacting throughput (or offsetting gains) |

#### ✅ Good: Normalize Throughput

```markdown
## Throughput — Sprint 14

Total PRs merged: 28
Team size: 7 developers
PRs per developer: 4.0

Baseline (pre-AI):
  Total PRs merged: 22
  Team size: 7 developers
  PRs per developer: 3.1

Change: +29% throughput per developer ✅

Context: Average PR size decreased from 340 to 210 lines,
suggesting more granular, reviewable changes.
```

#### ❌ Bad: Raw Counts Without Context

```
We merged 28 PRs this sprint! That's 6 more than last sprint!

(Last sprint had a holiday and one developer was on PTO.
Also, 8 of the 28 PRs were auto-generated dependency bumps.)
```

---

### 3. Time in Review

How long PRs wait for and go through code review. AI tools can affect this in two ways: AI pre-review catches issues before human review, and AI-generated code may need MORE careful review.

#### Tracking Template

```markdown
## Review Time Analysis — [Period]

| Metric | Before AI | After AI | Change |
|--------|-----------|----------|--------|
| Time to first review | 6.2 hrs | 4.1 hrs | ▼ 34% |
| Review rounds | 2.3 avg | 1.8 avg | ▼ 22% |
| Time in review | 18.4 hrs | 12.7 hrs | ▼ 31% |
| Review comments | 4.7 avg | 3.2 avg | ▼ 32% |

### Review Comment Type Breakdown
| Type | Before | After | Interpretation |
|------|--------|-------|----------------|
| Style/formatting | 1.8 | 0.4 | AI + linters handle this |
| Logic errors | 0.9 | 1.1 | Slightly more logic issues |
| Missing tests | 0.8 | 0.3 | AI generates tests |
| Documentation | 0.7 | 0.2 | AI documents code |
| Architecture | 0.5 | 1.3 | Reviewers focus on design |
```

#### ✅ Good: Track Review Quality Alongside Speed

```
Review time decreased 31%, AND review comments shifted from
style issues (AI handles) to architecture concerns (human judgment).
This means human reviewers are now focusing on higher-value feedback.
```

#### ❌ Bad: Only Track Review Speed

```
Review time decreased 31%! AI is making reviews faster!

(Reviews are faster because reviewers are rubber-stamping AI code
without actually understanding it.)
```

---

### 4. Task Completion Rate

Percentage of planned sprint tasks that are completed within the sprint.

#### ✅ Good: Track Completion with Confidence Data

```markdown
Sprint 14 Completion:
  Planned: 18 tasks (34 story points)
  Completed: 16 tasks (31 story points)
  Completion rate: 89% tasks, 91% points

  Pre-AI baseline:
    Average completion: 78% tasks, 75% points

  Improvement: +11% task completion, +16% point completion

  Note: Team reports higher confidence in sprint commitments
  due to AI assistance with routine tasks.
```

#### ❌ Bad: Inflate Estimates to Boost Completion

```
Completion rate: 100%!

(Team started sandbagging estimates — planning 20 points
when they could do 35 — to guarantee "100% completion with AI.")
```

---

### 5. Sprint Velocity Trend

Story points completed per sprint, tracked over time. Useful for spotting gradual improvement or decline.

```
Sprint Velocity Trend (Story Points)
─────────────────────────────────────
Sprint │ Points │ AI Status  │ Notes
───────┼────────┼────────────┼──────────────────
  8    │   28   │ No AI      │ Pre-baseline
  9    │   31   │ No AI      │ Pre-baseline
 10    │   29   │ No AI      │ Baseline avg: 29.3
 11    │   26   │ AI Week 1  │ Learning dip (expected)
 12    │   30   │ AI Week 3  │ Back to baseline
 13    │   34   │ AI Week 5  │ First real gain
 14    │   37   │ AI Week 7  │ Sustained improvement
 15    │   36   │ AI Week 9  │ Plateau — new normal?
```

#### ✅ Good: Show the Learning Curve

```
It's normal for velocity to DIP in the first 1-2 sprints after
AI adoption, then recover and exceed baseline by sprint 3-4.

Don't panic about the initial dip — it's the learning curve.
```

#### ❌ Bad: Expect Immediate Gains

```
"We adopted AI last sprint and velocity went DOWN. This isn't working.
Let's cancel the licenses."
```

---

### 6. Time Saved Per Task (Survey-Based)

Self-reported time savings for specific task categories. Valuable qualitative data to complement hard metrics.

#### Survey Template

```markdown
## Time Savings Survey — [Sprint/Month]

For each task type, estimate how much time AI tools saved
compared to doing the task without AI assistance.

| Task Type | AI Helped? | Estimated Time Saved | Confidence |
|-----------|-----------|---------------------|------------|
| Writing new functions | Yes/No | _____ min/hrs | High/Med/Low |
| Writing tests | Yes/No | _____ min/hrs | High/Med/Low |
| Bug fixing | Yes/No | _____ min/hrs | High/Med/Low |
| Code review prep | Yes/No | _____ min/hrs | High/Med/Low |
| Documentation | Yes/No | _____ min/hrs | High/Med/Low |
| Refactoring | Yes/No | _____ min/hrs | High/Med/Low |
| Learning new APIs | Yes/No | _____ min/hrs | High/Med/Low |
| Debugging | Yes/No | _____ min/hrs | High/Med/Low |
| Boilerplate/CRUD | Yes/No | _____ min/hrs | High/Med/Low |
| Complex algorithms | Yes/No | _____ min/hrs | High/Med/Low |

### Where AI was MOST helpful this sprint:
_____________________________________________

### Where AI was LEAST helpful or counterproductive:
_____________________________________________
```

---

## Anti-Metrics: What NOT to Use

### ❌ Lines of Code Per Day

```
Why this is harmful:
- AI generates verbose code. 500 LOC/day with AI ≠ better than 100 LOC/day without.
- Incentivizes accepting bloated AI suggestions without refactoring.
- The best code change is often REMOVING lines.
- A developer who deletes 200 lines of technical debt is more
  productive than one who adds 500 lines of boilerplate.
```

### ❌ AI Suggestion Acceptance Rate (as a KPI)

```
Why this is harmful:
- Turns a personal workflow tool into a surveillance metric.
- Developers start accepting bad suggestions to boost their "score."
- Low acceptance might mean a developer is working on complex tasks
  where AI is less useful — that's not a problem.
- Different languages and task types have wildly different acceptance
  rates (TypeScript: ~35%, C++: ~15%). Comparing is meaningless.
```

### ❌ Number of AI Conversations or Prompts

```
Why this is harmful:
- More prompts could mean the developer is struggling.
- Fewer prompts could mean they found AI unhelpful and stopped.
- Some developers get great results in 1 prompt; others iterate.
  Both approaches are valid.
```

### ✅ Instead: Use AI-Specific Metrics as CONTEXT, Not KPIs

```
Copilot acceptance rates by language and task type are useful for:
- Understanding which workflows benefit most from AI
- Identifying areas where AI training/prompts need improvement
- Informing tool selection and configuration

They are NOT useful for:
- Evaluating individual developer performance
- Justifying ROI (use business outcomes instead)
- Setting adoption targets ("everyone must accept 30%+")
```

---

## GitHub Copilot Metrics

### Copilot Business Dashboard

If your organization uses GitHub Copilot Business or Enterprise, you have access to usage analytics.

#### Available Metrics

| Metric | What It Tells You | How to Use It |
|--------|------------------|---------------|
| Active users | How many devs use Copilot regularly | Adoption tracking |
| Acceptance rate | % of suggestions accepted | Tool effectiveness by language |
| Lines suggested | Volume of AI-generated code | Engagement level |
| Languages used | Which languages get most AI use | Target training and prompts |
| Time saved (est.) | GitHub's estimate of time saved | Directional indicator only |

#### ✅ Good: Use Dashboard for Team-Level Insights

```markdown
## Copilot Usage Report — March 2025

### Adoption
- Licensed seats: 25
- Active users (last 30 days): 22 (88% adoption)
- Daily active users: 15 (60% daily engagement)

### Effectiveness by Language
| Language | Acceptance Rate | Interpretation |
|----------|----------------|----------------|
| TypeScript | 38% | Strong — good prompt/context |
| Python | 35% | Strong — well-suited for AI |
| Go | 22% | Moderate — more structured language |
| Rust | 12% | Low — complex ownership model |
| SQL | 41% | Very strong — pattern-heavy |

### Insight
AI is most effective for pattern-heavy languages (TS, Python, SQL).
Consider additional prompt engineering training for Go and Rust teams.
```

#### ❌ Bad: Use Dashboard for Individual Performance

```
Developer scoreboard:
  Alice: 45% acceptance — GREAT
  Bob: 12% acceptance — NEEDS IMPROVEMENT
  Carol: 8% acceptance — NOT USING AI

(This is surveillance, not measurement.)
```

---

## Metric Collection Automation

### GitHub Actions Workflow: Weekly Metrics

```yaml
# .github/workflows/productivity-metrics.yml
name: Weekly Productivity Metrics

on:
  schedule:
    - cron: '0 8 * * 1'  # Every Monday at 8 AM UTC
  workflow_dispatch:

permissions:
  contents: read
  pull-requests: read

jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install Dependencies
        run: pip install requests

      - name: Collect PR Metrics
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          python scripts/collect_cycle_time.py \
            --repo ${{ github.repository }} \
            --weeks 1

      - name: Generate Summary
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const files = fs.readdirSync('.').filter(f =>
              f.startsWith('cycle-time-report')
            );

            if (files.length === 0) {
              console.log('No report file generated');
              return;
            }

            const report = JSON.parse(fs.readFileSync(files[0], 'utf8'));
            const summary = report.summary;

            const body = `## Weekly Productivity Metrics

            | Metric | Value |
            |--------|-------|
            | PRs Merged | ${summary.period_prs} |
            | Avg Cycle Time | ${summary.avg_cycle_days} days |
            | Median Cycle Time | ${summary.median_cycle_days} days |
            | P90 Cycle Time | ${summary.p90_cycle_days} days |

            *Auto-generated on ${new Date().toISOString().split('T')[0]}*`;

            console.log(body);

      - name: Upload Report
        uses: actions/upload-artifact@v4
        with:
          name: productivity-metrics-${{ github.run_id }}
          path: cycle-time-report-*.json
          retention-days: 90
```

### Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "title": "AI Productivity Metrics",
    "tags": ["ai-metrics", "productivity"],
    "panels": [
      {
        "title": "PR Cycle Time (days)",
        "type": "timeseries",
        "gridPos": { "h": 8, "w": 12, "x": 0, "y": 0 },
        "targets": [
          {
            "expr": "avg(pr_cycle_time_days) by (week)",
            "legendFormat": "Average Cycle Time"
          },
          {
            "expr": "quantile(0.9, pr_cycle_time_days) by (week)",
            "legendFormat": "P90 Cycle Time"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                { "color": "green", "value": null },
                { "color": "yellow", "value": 3 },
                { "color": "red", "value": 5 }
              ]
            }
          }
        }
      },
      {
        "title": "PR Throughput (per developer per sprint)",
        "type": "stat",
        "gridPos": { "h": 4, "w": 6, "x": 12, "y": 0 },
        "targets": [
          {
            "expr": "sum(prs_merged_weekly) / count(distinct(pr_author))",
            "legendFormat": "PRs/Dev/Week"
          }
        ]
      },
      {
        "title": "Sprint Velocity Trend",
        "type": "timeseries",
        "gridPos": { "h": 8, "w": 12, "x": 0, "y": 8 },
        "targets": [
          {
            "expr": "sprint_velocity_points",
            "legendFormat": "Story Points"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "custom": {
              "drawStyle": "line",
              "lineWidth": 2,
              "pointSize": 5,
              "showPoints": "always"
            }
          }
        }
      },
      {
        "title": "Review Time Breakdown",
        "type": "barchart",
        "gridPos": { "h": 8, "w": 12, "x": 12, "y": 8 },
        "targets": [
          {
            "expr": "avg(time_to_first_review_hours)",
            "legendFormat": "Time to First Review"
          },
          {
            "expr": "avg(review_duration_hours)",
            "legendFormat": "Review Duration"
          }
        ]
      }
    ]
  }
}
```

---

## Productivity Metrics Reporting Template

### Monthly Productivity Report

```markdown
# Productivity Metrics Report — [Month Year]

## Executive Summary
Overall productivity trend: [ ▲ Improving / ► Stable / ▼ Declining ]
AI contribution assessment: [ High / Moderate / Low / Uncertain ]

## Key Metrics

### Velocity
| Metric | Baseline | Last Month | This Month | Trend |
|--------|----------|-----------|------------|-------|
| Avg cycle time | ___ days | ___ days | ___ days | |
| Median cycle time | ___ days | ___ days | ___ days | |
| P90 cycle time | ___ days | ___ days | ___ days | |
| PRs merged | ___ | ___ | ___ | |
| PRs per dev | ___ | ___ | ___ | |
| Sprint velocity | ___ pts | ___ pts | ___ pts | |

### Review Efficiency
| Metric | Baseline | Last Month | This Month | Trend |
|--------|----------|-----------|------------|-------|
| Time to first review | ___ hrs | ___ hrs | ___ hrs | |
| Review duration | ___ hrs | ___ hrs | ___ hrs | |
| Review rounds | ___ | ___ | ___ | |
| Comments per PR | ___ | ___ | ___ | |

### Developer Self-Report (Survey)
| Question | Avg Score (1-5) | Trend |
|----------|----------------|-------|
| AI helps me be faster | ___ | |
| AI helps with routine tasks | ___ | |
| I'd be less productive without AI | ___ | |

### Time Savings by Task Type (Survey)
| Task Type | Avg Savings | # Responses | Confidence |
|-----------|------------|-------------|------------|
| New features | ___ hrs/wk | ___ | |
| Writing tests | ___ hrs/wk | ___ | |
| Bug fixes | ___ hrs/wk | ___ | |
| Documentation | ___ hrs/wk | ___ | |
| Code review | ___ hrs/wk | ___ | |

## Analysis
### What's Working
1. ___________________________________________
2. ___________________________________________

### What Needs Attention
1. ___________________________________________
2. ___________________________________________

## Actions for Next Month
1. ___________________________________________
2. ___________________________________________
```

---

## Common Productivity Measurement Mistakes

### ✅ Do: Measure Outcomes, Not Activity

```
Good: "How many features shipped and at what quality?"
Bad:  "How many hours were spent coding?"
```

### ❌ Don't: Compare Developers to Each Other

```
Comparing Alice's AI-boosted metrics to Bob's is toxic.
Compare the TEAM's performance over time instead.
```

### ✅ Do: Account for Task Complexity

```
10 story points of CRUD endpoints ≠ 10 story points of algorithm design.
AI helps more with the former. Normalize for complexity.
```

### ❌ Don't: Ignore Rework

```
"We shipped 8 features this sprint!"
(But 3 had to be reworked next sprint due to bugs.)
Net features: 5, plus rework cost.
```

### ✅ Do: Include the Full Cost of Speed

```
If AI helps you ship 30% faster but creates 20% more bugs,
the NET productivity gain is much less than 30%.
Always pair velocity metrics with quality metrics.
```

### ❌ Don't: Measure During Disruptions

```
Sprint 15 had a production incident, 2 people on vacation,
and a major refactor. Don't use it as a comparison point.
Exclude anomalous sprints from trend analysis.
```

---

## Actionable Checklist: Setting Up Productivity Metrics

- [ ] **Define your metrics** — Select 4–6 metrics from this guide
- [ ] **Capture baselines** — 4–6 weeks of pre-AI data
- [ ] **Automate collection** — GitHub Actions, scripts, or API integrations
- [ ] **Set up a dashboard** — Grafana, Google Sheets, or team wiki
- [ ] **Establish cadence** — Weekly quick check, monthly deep analysis
- [ ] **Design surveys** — Monthly developer self-report on time savings
- [ ] **Define thresholds** — What's "good" vs. "needs attention" for each metric
- [ ] **Share transparently** — Results visible to the whole team
- [ ] **Review quarterly** — Are we measuring the right things?
- [ ] **Pair with quality** — Never report velocity without quality alongside it

---

## Next Steps

- [Measuring AI Impact](measuring-ai-impact.md) — The overall framework for AI measurement
- [Quality Metrics](quality-metrics.md) — Pair velocity data with quality tracking
- [ROI Analysis](roi-analysis.md) — Turn productivity data into business justification
- [Feedback Loops](../09-best-practices/feedback-loops.md) — Use metrics to drive continuous improvement
- [Metrics Theater](../10-anti-patterns/metrics-theater.md) — Avoid common measurement anti-patterns
- [AI Code Review Workflow](../04-review-processes/ai-code-review-workflow.md) — How review efficiency connects to productivity
- [Cross-Team Consistency](../05-scaling-ai-across-teams/cross-team-consistency.md) — Consistent metrics across teams
- [Rollout Strategy](../05-scaling-ai-across-teams/rollout-strategy.md) — Measuring impact during phased adoption
