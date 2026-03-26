# Compliance & Audit Trails with SDD

> Using spec-driven development for regulated industries and compliance requirements.

---

## Why SDD Is a Natural Fit for Compliance

Regulated industries require:
- **Traceability**: Requirement → Implementation → Verification
- **Documentation**: Evidence of design decisions
- **Review records**: Proof of human oversight
- **Change management**: Tracked modifications with rationale

SDD provides all of this by default when practiced correctly.

---

## The Traceability Chain

```
Regulation ──► Requirement ──► Spec ──► Plan ──► Task ──► Code ──► Test
   │              │              │        │        │        │        │
   └──── Every link is documented, version-controlled, and reviewable ────┘
```

### Example: GDPR Data Portability

```
GDPR Art. 20
  └► Req: "Users can export their data in machine-readable format"
      └► Spec: .specify/specs/data-export/spec.md (AC-1 through AC-5)
          └► Plan: plan.md §3 (JSON export via async job)
              └► Task: tasks.md #4 (Implement export endpoint)
                  └► Code: src/api/export.ts
                      └► Test: tests/api/export.test.ts (verifies AC-1 through AC-5)
```

---

## Compliance Spec Template

```markdown
# Compliance Feature: [Name]

## Regulatory Reference
| Regulation | Section | Requirement |
|-----------|---------|-------------|
| [Reg name] | [Article/Section] | [Exact requirement text] |

## Compliance Mapping
| Regulation | Acceptance Criteria | Verification Method |
|-----------|-------------------|-------------------|
| GDPR Art. 20 | AC-1: Export in JSON format | Automated test |
| GDPR Art. 15 | AC-2: Delivered within 72h | Integration test |
| SOC 2 CC6.1 | AC-3: Requires 2FA auth | Security review |

## Risk Assessment
| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Export includes unauthorized data | Low | Critical | Row-level security |
| Export exceeds timeout | Medium | Medium | Async processing |

## Audit Requirements
- All spec changes tracked in git with author and date
- All reviews documented in PR comments
- All test results stored in CI/CD artifacts
- Quarterly compliance review of spec accuracy
```

---

## Audit-Ready Artifacts

### What Auditors Want to See

| Artifact | SDD Provides | Location |
|----------|-------------|----------|
| Requirements documentation | spec.md | .specify/specs/ |
| Design documentation | plan.md | .specify/specs/ |
| Change history | Git log | git log --follow spec.md |
| Review records | PR reviews | GitHub PR comments |
| Test evidence | CI/CD results | GitHub Actions logs |
| Traceability matrix | Spec ↔ Code references | Commit messages + spec links |

### Generating an Audit Report

```bash
# List all specs and their status
find .specify/specs -name "spec.md" -exec head -5 {} \;

# Show review history for a spec
git log --follow --format="%H %ad %an: %s" .specify/specs/data-export/spec.md

# Find all commits referencing a spec
git log --grep="data-export/spec" --oneline

# Verify all acceptance criteria have tests
grep -r "AC-" tests/ | sort
```

---

## Industry-Specific Patterns

### Healthcare (HIPAA)
```markdown
## Required Spec Sections for HIPAA Features
- PHI (Protected Health Information) data flow diagram
- Access control requirements by role
- Encryption requirements (at rest and in transit)
- Audit logging requirements for PHI access
- Breach notification procedure
```

### Finance (SOX/PCI-DSS)
```markdown
## Required Spec Sections for Financial Features
- Transaction integrity requirements
- Segregation of duties (who can do what)
- Data retention and purge policies
- Audit trail requirements
- Encryption standards for payment data
```

### Government (FedRAMP)
```markdown
## Required Spec Sections for Government Features
- Security control mapping (NIST 800-53)
- Authorization boundary definition
- Continuous monitoring requirements
- Incident response procedures
```

---

## Automating Compliance Checks

### CI/CD Pipeline
```yaml
compliance-check:
  steps:
    - name: Verify spec exists for changed features
      run: |
        # Check if changed src/ directories have corresponding specs
        for dir in $(git diff --name-only origin/main -- 'src/features/'); do
          feature=$(echo $dir | cut -d/ -f3)
          if [ ! -f ".specify/specs/$feature/spec.md" ]; then
            echo "ERROR: No spec for feature $feature"
            exit 1
          fi
        done

    - name: Verify regulatory mapping
      run: |
        # Check compliance specs have regulatory references
        for spec in .specify/specs/*/spec.md; do
          if grep -q "compliance" <<< "$spec"; then
            if ! grep -q "Regulatory Reference" "$spec"; then
              echo "WARN: Compliance spec missing regulatory reference: $spec"
            fi
          fi
        done

    - name: Verify acceptance criteria coverage
      run: |
        # Check all ACs have corresponding tests
        speckit coverage --report compliance-report.md
```

---

## The Compliance SDD Checklist

| Phase | Compliance Action | Evidence |
|-------|------------------|----------|
| Specify | Map requirements to regulations | Regulatory reference table in spec |
| Plan | Security and risk assessment | Risk table in plan.md |
| Tasks | Include compliance verification tasks | Specific compliance test tasks |
| Implement | Follow secure coding guidelines | Code review with security checklist |
| Verify | Run compliance acceptance tests | Test results in CI artifacts |
| Archive | Store all artifacts with retention | Git tags + archive policy |

---

*Next: [Full-Stack Web App Spec →](../08-domain-examples/fullstack-web-app.md)*
