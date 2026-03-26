# Enterprise & Organization Strategies

> Scaling Copilot instructions across teams, repositories, and the organization.

---

## The Enterprise Challenge

| Scale | Challenge |
|-------|-----------|
| 1 repo, 1 team | Keep instructions accurate |
| 10 repos, 3 teams | Maintain consistency across repos |
| 100+ repos, 10+ teams | Standardize without stifling team autonomy |

---

## Governance Architecture

### Three-Tier Model

```
┌─────────────────────────────────────────────┐
│  Tier 1: Organization Policies               │
│  Who: Platform/Security team                 │
│  What: Security rules, universal standards   │
│  Where: GitHub Org Settings → Copilot        │
├─────────────────────────────────────────────┤
│  Tier 2: Repository Standards                │
│  Who: Team leads                             │
│  What: Stack, conventions, commands          │
│  Where: .github/copilot-instructions.md      │
├─────────────────────────────────────────────┤
│  Tier 3: Domain Rules                         │
│  Who: Domain experts                         │
│  What: Path-specific patterns, workflows     │
│  Where: .github/instructions/*.instructions.md│
└─────────────────────────────────────────────┘
```

---

## Org-Level Instruction Strategy

### What to Put at Org Level

Only rules that apply to **100% of repositories**:

```markdown
## Security (Universal)
- Never commit secrets, credentials, or API keys
- Use parameterized queries for all database operations
- Validate and sanitize all user input

## Quality (Universal)
- All code changes must include tests
- Use conventional commit format
- All public APIs must have documentation
```

### What NOT to Put at Org Level

- Framework-specific rules (not all repos use React)
- Package manager preferences (not all repos use pnpm)
- Deployment commands (differ per repo)
- Detailed coding conventions (vary by language/team)

---

## Template Library

Maintain a central repository of instruction file templates:

```
org-copilot-templates/
├── README.md                          # Usage guide
├── base/
│   └── copilot-instructions.md        # Minimal starter
├── stacks/
│   ├── nextjs-typescript.md           # Next.js + TS conventions
│   ├── express-typescript.md          # Express + TS conventions
│   ├── react-native.md               # React Native conventions
│   └── python-fastapi.md             # Python + FastAPI conventions
├── domains/
│   ├── frontend.instructions.md       # Generic frontend rules
│   ├── api.instructions.md            # Generic API rules
│   ├── database.instructions.md       # Generic DB rules
│   └── testing.instructions.md        # Generic testing rules
└── prompts/
    ├── add-feature.prompt.md
    ├── fix-bug.prompt.md
    └── generate-tests.prompt.md
```

### Usage

1. Team creates a new repo
2. Copies relevant templates from the central repo
3. Customizes for their specific project
4. Maintains locally going forward

---

## Compliance and Audit

### Automated Compliance Check

```yaml
# .github/workflows/copilot-compliance.yml
name: Copilot Instructions Compliance
on:
  pull_request:
    paths: ['.github/copilot-instructions.md', '.github/instructions/**', 'AGENTS.md']

jobs:
  compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check required sections
        run: |
          file=".github/copilot-instructions.md"
          for section in "## Commands" "## Conventions" "## Do Not"; do
            if ! grep -q "$section" "$file" 2>/dev/null; then
              echo "::error::Missing required section: $section"
              exit 1
            fi
          done
      
      - name: Check file size
        run: |
          size=$(wc -w < .github/copilot-instructions.md)
          if [ "$size" -gt 1500 ]; then
            echo "::warning::copilot-instructions.md exceeds 1500 words ($size words)"
          fi
      
      - name: Security scan
        run: |
          if grep -rP '[\x{200B}-\x{200F}\x{FEFF}]' .github/ 2>/dev/null; then
            echo "::error::Hidden Unicode characters detected!"
            exit 1
          fi
```

### Quarterly Audit Dashboard

Track across repositories:

| Repository | Has Instructions | Last Updated | Word Count | Has Commands | Has Tests |
|-----------|-----------------|--------------|------------|-------------|-----------|
| web-app | ✅ | 2 weeks ago | 487 | ✅ | ✅ |
| api-service | ✅ | 3 months ago | 1,240 ⚠️ | ✅ | ✅ |
| mobile-app | ❌ | N/A | N/A | ❌ | ❌ |
| data-pipeline | ✅ | 6 months ago ⚠️ | 312 | ✅ | ❌ |

---

## Rollout Plan

### Phase 1: Pilot (Week 1-2)

1. Choose 2-3 repositories with engaged tech leads
2. Create copilot-instructions.md for each
3. Gather feedback from developers
4. Iterate on content and format

### Phase 2: Template (Week 3-4)

1. Extract common patterns into templates
2. Create the central template repository
3. Document the process for creating instruction files
4. Write an internal guide

### Phase 3: Expand (Week 5-8)

1. Communicate initiative to all teams
2. Provide templates and support
3. Tech leads create instructions for their repos
4. Platform team reviews and provides feedback

### Phase 4: Standardize (Week 9-12)

1. Set organization-level instructions
2. Establish CODEOWNERS for instruction files
3. Create CI compliance checks
4. Schedule quarterly audits

### Phase 5: Optimize (Ongoing)

1. Measure effectiveness (developer survey)
2. Refine templates based on real usage
3. Share best practices across teams
4. Update as Copilot features evolve

---

## Metrics to Track

| Metric | How to Measure | Target |
|--------|---------------|--------|
| Adoption | % of repos with instruction files | > 80% |
| Freshness | Average age of last instruction update | < 3 months |
| Size compliance | % of files under word limit | > 90% |
| Required sections | % of files with Commands section | 100% |
| Developer satisfaction | Quarterly survey | > 4/5 |
| Copilot quality | Fewer revisions needed on AI-generated PRs | Decreasing trend |

---

## Internal Communication Template

### Announcement Email

```
Subject: Standardizing AI Development Assistance with Copilot Instructions

Team,

We're rolling out standardized Copilot instruction files across our repos.

What: A `.github/copilot-instructions.md` file in each repository that tells
Copilot how to work with our code — our conventions, commands, and boundaries.

Why: AI tools work dramatically better with project-specific context.
Teams with good instructions report 30-50% less time spent fixing AI suggestions.

Your action: [Link to template repo]. Choose a template for your stack,
customize it, and add it to your repo by [date].

Support: #copilot-instructions Slack channel.

Questions? Reach out to the platform team.
```

---

*Next: [Frontend & React Domain Guide](../06-domain-guides/frontend-react.md) →*
