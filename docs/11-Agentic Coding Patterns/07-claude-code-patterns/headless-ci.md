# Headless and CI Mode

> Running Claude Code non-interactively for automation — single-shot queries in scripts, automated PR reviews, issue triage in CI/CD, and event-driven workflows.

---

## Overview

Claude Code doesn't require a human at the keyboard. In headless mode, it processes tasks programmatically — perfect for CI/CD pipelines, scheduled automation, and event-driven workflows.

```
┌──────────────────────────────────────────────────────────────┐
│              Claude Code Execution Modes                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Interactive            Headless / CI                       │
│   ───────────            ────────────                        │
│   Human at terminal      Script or pipeline                  │
│   Conversational         Single-shot or batch                │
│   Approve tool calls     Pre-approved tools                  │
│                                                              │
│   claude                 claude --print "task"               │
│                          claude -p "task"                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Single-Shot Queries with --print

The `--print` (or `-p`) flag runs Claude Code non-interactively and outputs the result:

```bash
# Get a quick answer
claude -p "What testing framework does this project use?"

# Generate code and pipe to a file
claude -p "Write a utility function that validates email addresses" > utils.ts

# Use in shell scripts
FRAMEWORK=$(claude -p "What package manager does this project use?")
echo "This project uses: $FRAMEWORK"

# Analyze code
claude -p "Find potential security issues in src/auth/" | tee security-report.txt
```

## GitHub Actions Integration

### Automated PR Review

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Review PR
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          DIFF=$(git diff origin/main...HEAD)
          claude -p "Review this PR diff. Focus on bugs, security issues,
          and logic errors. Output as markdown.

          $DIFF" > review.md

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: review
            });
```

### Automated Issue Triage

```yaml
# .github/workflows/claude-triage.yml
name: Issue Triage

on:
  issues:
    types: [opened]

jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Analyze Issue
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p "Analyze this GitHub issue and suggest:
          1. Priority (P0-P3)
          2. Component labels
          3. Whether it's a bug, feature, or question
          4. Relevant files to investigate

          Title: ${{ github.event.issue.title }}
          Body: ${{ github.event.issue.body }}" > triage.md

      - name: Add Labels
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const triage = fs.readFileSync('triage.md', 'utf8');
            // Parse triage output and apply labels
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Auto-Triage\n${triage}`
            });
```

## GitLab CI/CD Integration

```yaml
# .gitlab-ci.yml
claude-review:
  stage: review
  image: node:20
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  script:
    - npm install -g @anthropic-ai/claude-code
    - |
      DIFF=$(git diff origin/$CI_MERGE_REQUEST_TARGET_BRANCH_NAME...HEAD)
      claude -p "Review this merge request diff for bugs and issues:
      $DIFF" > review.md
    - cat review.md
  artifacts:
    paths:
      - review.md
```

## Use Cases

### 1. Automated Code Review on Every PR

```
┌────────────────────────────────────────────────┐
│         PR Review Automation                    │
├────────────────────────────────────────────────┤
│                                                 │
│   PR Opened ──▶ Claude analyzes diff            │
│                      │                          │
│                      ▼                          │
│              Posts review comment                │
│              with findings                       │
│                      │                          │
│                      ▼                          │
│              Developer reviews                   │
│              Claude's feedback                   │
│                      │                          │
│                      ▼                          │
│              Addresses issues                    │
│              or dismisses                         │
│                                                 │
└────────────────────────────────────────────────┘
```

### 2. Nightly Code Quality Analysis

```yaml
# Run every night at 2 AM
on:
  schedule:
    - cron: '0 2 * * *'

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      - name: Analyze Code Quality
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p "Analyze the codebase for:
          1. Dead code and unused exports
          2. Functions over 50 lines
          3. Missing error handling
          4. TODO/FIXME comments that should be issues
          Output as a structured report." > quality-report.md
      - name: Upload Report
        uses: actions/upload-artifact@v4
        with:
          name: quality-report
          path: quality-report.md
```

### 3. Automated Documentation Updates

```bash
#!/bin/bash
# scripts/update-docs.sh

# Generate API docs from code
claude -p "Read all route handlers in src/api/ and generate
API documentation in OpenAPI 3.0 format. Include request/response
schemas, status codes, and descriptions." > docs/api/openapi.yaml

# Update README with current project structure
claude -p "Update the Project Structure section of README.md
to reflect the current directory layout. Keep all other sections
unchanged." --allow-all-tools
```

### 4. Issue-to-PR Automation

```yaml
# When an issue gets the "auto-implement" label
on:
  issues:
    types: [labeled]

jobs:
  implement:
    if: github.event.label.name == 'auto-implement'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      - name: Implement Issue
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          git checkout -b auto/${{ github.event.issue.number }}
          claude -p "Implement this issue:
          Title: ${{ github.event.issue.title }}
          Body: ${{ github.event.issue.body }}
          Follow all conventions in CLAUDE.md.
          Run tests after implementation." --allow-all-tools
          git push origin auto/${{ github.event.issue.number }}
      - name: Create PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.pulls.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Auto: ${{ github.event.issue.title }}`,
              head: `auto/${{ github.event.issue.number }}`,
              base: 'main',
              body: `Closes #${{ github.event.issue.number }}\n\nAutomated implementation by Claude Code.`
            });
```

## The Channels Feature

Push events from external systems into Claude Code sessions:

```
┌──────────────────────────────────────────────────┐
│               Channels                            │
├──────────────────────────────────────────────────┤
│                                                   │
│   Telegram ──┐                                    │
│              │                                    │
│   Discord ───┤──▶ Claude Code Session             │
│              │                                    │
│   Webhooks ──┘                                    │
│                                                   │
│   Use cases:                                      │
│   • Push alerts into a debugging session          │
│   • Send customer reports for investigation       │
│   • Trigger actions from team chat                │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Remote Control

Continue local Claude Code sessions from mobile or another device — useful for monitoring long-running tasks or providing input while away from your desk.

## Best Practices for CI Integration

| Practice | Why |
|---|---|
| **Pin Claude Code version** | Reproducible builds |
| **Set timeout limits** | Prevent runaway costs |
| **Use secrets for API keys** | Never hardcode credentials |
| **Cache installations** | Faster pipeline execution |
| **Limit tool access** | CI should not modify infra |
| **Review outputs** | Automated ≠ unsupervised |
