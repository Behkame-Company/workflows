# PR Templates for AI-Assisted Work

> Templates that support transparency about AI involvement. When reviewers know code is AI-generated, they adjust their review depth and focus — leading to better outcomes for everyone.

---

## Why AI Disclosure Matters

Reviewers approach AI-generated code differently than human-written code:

| Code Type | Review Focus |
|---|---|
| Human-written | Intent, style, logic, patterns |
| AI-generated (fully) | Correctness, edge cases, hallucinations, security |
| AI-assisted (partially) | Integration seams, consistency, human+AI boundary |

Without disclosure, reviewers may:
- ❌ Apply light "style review" to AI code that needs deep logic review
- ❌ Spend time on style feedback that AI already got right
- ❌ Miss hallucinated imports or APIs because they assume the developer verified
- ❌ Not know to check for common AI failure modes

---

## Standard PR Template

### Template File

Save this as `.github/PULL_REQUEST_TEMPLATE.md` in your repository:

```markdown
## Summary

<!-- What does this PR do? Brief description of the changes. -->

## AI Assistance

### Was AI used in this PR?
<!-- Select one -->
- [ ] No AI assistance
- [ ] AI autocomplete only (inline suggestions)
- [ ] AI-generated code (partial — specific functions or sections)
- [ ] AI-generated code (full — entire feature or module)
- [ ] AI-assisted refactoring
- [ ] AI-generated tests

### AI Tool Used
<!-- Which tool(s) were used? -->
- [ ] GitHub Copilot (autocomplete)
- [ ] GitHub Copilot Chat / CLI
- [ ] Claude Code
- [ ] Other: _____________

### Prompt Summary
<!-- If AI generated significant code, briefly describe what you asked for -->

### Human Modifications
<!-- What did you change, add, or fix after AI generation? -->

## Review Focus Areas

<!-- What areas need the most attention from reviewers? -->
- [ ] Business logic correctness
- [ ] Security implications
- [ ] Error handling and edge cases
- [ ] Performance considerations
- [ ] Integration with existing code
- [ ] Other: _____________

## Testing

### Test Status
- [ ] New tests added
- [ ] Existing tests pass
- [ ] Manual testing completed
- [ ] Edge cases tested

### Test Coverage
<!-- What's the coverage for new/changed code? -->

## Security Considerations

- [ ] No new security-sensitive code
- [ ] Input validation added/updated
- [ ] Authentication/authorization verified
- [ ] No secrets or credentials in code
- [ ] SAST scan passes

## Checklist

- [ ] Code follows team style guide
- [ ] Tests pass locally and in CI
- [ ] SAST scan is clean
- [ ] I have reviewed all AI-generated code line by line
- [ ] No new dependencies added (or dependencies approved)
- [ ] Documentation updated if needed
- [ ] PR size is reviewable (< 500 lines, or split explanation below)
```

---

## Scenario-Specific Templates

### For Fully AI-Generated Features

```markdown
## AI-Generated Feature: [Feature Name]

### What was generated
<!-- Describe the feature and which files were AI-generated -->

### Generation approach
- **Tool:** [Copilot CLI / Claude Code / etc.]
- **Prompt strategy:** [Single prompt / iterative / multi-step]
- **Prompt(s) used:**
  ```
  [Include the key prompts used]
  ```

### What I verified
- [ ] Read every generated line
- [ ] Tested happy path manually
- [ ] Tested error paths manually
- [ ] Checked for hallucinated imports/APIs
- [ ] Verified against project patterns
- [ ] Security review completed

### What I changed from AI output
<!-- List specific modifications you made -->
1. 
2. 
3. 

### Known limitations
<!-- Anything the reviewer should know -->
```

### For AI-Generated Tests

```markdown
## AI-Generated Tests: [Module/Feature]

### What's being tested
<!-- Which code are these tests covering? -->

### Generation details
- **Tool:** [Tool name]
- **Test framework:** [Vitest / Jest / etc.]
- **Coverage target:** [X%]

### Test quality verification
- [ ] Tests verify behavior, not implementation
- [ ] Edge cases covered (null, empty, boundary)
- [ ] Error paths tested
- [ ] No tautological tests
- [ ] Tests are independent (no shared state)
- [ ] Test descriptions are accurate

### Manual additions
<!-- Tests you added manually that AI missed -->
```

### For AI-Assisted Refactoring

```markdown
## AI-Assisted Refactoring: [Description]

### Motivation
<!-- Why is this refactoring needed? -->

### AI's role
- **Tool:** [Tool name]
- **Scope:** [What the AI refactored]
- **Strategy:** [How you guided the refactoring]

### Behavior preservation
- [ ] All existing tests pass
- [ ] No functional changes intended
- [ ] Manual smoke test completed
- [ ] Public API unchanged

### Changes made
<!-- Highlight the key structural changes -->
```

---

## GitHub PR Template Configuration

### Single Template (Recommended for Most Teams)

Place in `.github/PULL_REQUEST_TEMPLATE.md` — GitHub auto-loads it for every PR.

### Multiple Templates (For Diverse PR Types)

```
.github/
├── PULL_REQUEST_TEMPLATE/
│   ├── default.md           # Standard PR
│   ├── ai-generated.md      # Fully AI-generated PR
│   ├── ai-tests.md          # AI-generated tests
│   └── ai-refactor.md       # AI-assisted refactoring
```

Developers select the template when creating a PR via the GitHub UI or by appending `?template=ai-generated.md` to the PR creation URL.

---

## Encouraging Honest Disclosure

The goal is transparency, not punishment. Teams that make disclosure easy and judgment-free get better results:

✅ **Good culture:**
- AI disclosure is a normal checkbox, not a confession
- Reviewers use disclosure to focus their review effectively
- No stigma attached to AI-generated code
- Disclosure helps the team learn what works

❌ **Bad culture:**
- AI disclosure feels like admitting you didn't "really" write the code
- Developers hide AI usage to avoid scrutiny
- Reviewers treat AI-generated PRs as lower quality by default
- Disclosure creates extra bureaucracy without benefit

### Tips for Encouraging Disclosure

1. **Lead by example** — Team leads should disclose their own AI usage
2. **Make it easy** — Checkboxes are faster than free-form explanations
3. **Show the benefit** — "I knew to check edge cases because you flagged it as AI-generated"
4. **Never penalize** — Disclosure should never lead to negative consequences
5. **Celebrate learning** — When disclosure leads to catching an issue, celebrate the process

---

## Template Maintenance

### Quarterly Review

- [ ] Template sections are still relevant
- [ ] AI tool options are up to date
- [ ] Checklist items reflect current standards
- [ ] Team members find the template useful (not burdensome)
- [ ] New AI-related review learnings incorporated

### Feedback Collection

Add to your quarterly retro:
- *"Is the PR template helping reviewers focus?"*
- *"Is the AI disclosure section easy to fill out?"*
- *"What's missing from the template?"*
- *"What should we remove (too much overhead)?"*
