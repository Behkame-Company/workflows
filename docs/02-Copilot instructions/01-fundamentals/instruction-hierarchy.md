# Instruction Hierarchy & Precedence

> Understanding which instructions win when multiple levels exist.

---

## The Four Levels

Copilot instructions exist at four levels, from broadest to most specific:

```
┌─────────────────────────────────────┐
│  Level 1: Organization (broadest)   │  Set by org admins on GitHub.com
├─────────────────────────────────────┤
│  Level 2: Repository-wide           │  .github/copilot-instructions.md
├─────────────────────────────────────┤
│  Level 3: Path-specific             │  .github/instructions/*.instructions.md
├─────────────────────────────────────┤
│  Level 4: Personal (most specific)  │  User's personal Copilot settings
└─────────────────────────────────────┘
```

---

## Precedence Rules

**Priority order (highest → lowest):**

1. **Personal** custom instructions (set in IDE settings or GitHub.com)
2. **Path-specific** instructions (`.instructions.md` matching current file)
3. **Repository-wide** instructions (`copilot-instructions.md`)
4. **Organization** custom instructions (set by org admin)

### Key Behavior: Merging, Not Replacing

All applicable levels are **combined** and sent to the model together. They don't override each other — they stack. Only in cases of **direct contradiction** does priority matter.

```
Example: Working on src/components/Button.tsx

Copilot receives:
  + Organization instructions: "Always use TypeScript"
  + Repository instructions: "Use Next.js App Router patterns"
  + Path-specific (frontend.instructions.md): "Use Tailwind CSS for styling"
  + Personal: "Prefer verbose comments"

All four sets are merged into a single context.
```

---

## What Each Level Is For

### Organization Level

**Owner**: Platform team / engineering leadership
**Scope**: All repositories in the organization
**Set via**: GitHub.com → Organization Settings → Copilot → Custom Instructions

**Best used for:**
- Universal language requirements ("Always use TypeScript")
- Security policies ("Never commit secrets")
- Documentation standards ("All public functions need JSDoc")
- License compliance rules

**Example:**
```markdown
All code must be written in TypeScript with strict mode enabled.
Never commit secrets, credentials, or API keys to source control.
All public API functions must have JSDoc documentation.
Follow the organization's ESLint configuration.
```

⚠️ Keep organization instructions **minimal and universal**. If a rule doesn't apply to every repo, it doesn't belong here.

### Repository Level

**Owner**: Tech lead / repository maintainer
**Scope**: Everything in this one repository
**File**: `.github/copilot-instructions.md`

**Best used for:**
- Project-specific stack and patterns
- Build/test/lint commands
- Project structure overview
- Repo-specific conventions

### Path-Specific Level

**Owner**: Domain expert / team within the project
**Scope**: Files matching the `applyTo` glob pattern
**File**: `.github/instructions/NAME.instructions.md`

**Best used for:**
- Frontend vs. backend conventions
- Test file patterns
- Database migration safety rules
- Security-sensitive code requirements

### Personal Level

**Owner**: Individual developer
**Scope**: All repositories for this one person
**Set via**: IDE settings or GitHub.com personal settings

**Best used for:**
- Personal coding preferences (comment style, verbosity)
- Accessibility needs
- Language preferences for explanations

⚠️ Personal instructions can **conflict** with team standards. Use with caution.

---

## Conflict Resolution

### When Instructions Agree

```
Org:   "Use TypeScript"
Repo:  "Use TypeScript strict mode"
Path:  "Use TypeScript with React 18 patterns"

→ Copilot applies ALL — they're complementary (general → specific)
```

### When Instructions Conflict

```
Org:    "Use CSS Modules for styling"
Repo:   "Use Tailwind CSS for styling"

→ Copilot receives BOTH, but repo-level has higher priority
→ Result is unpredictable — Copilot may mix approaches
```

**Solution**: Avoid contradictions. If a repo needs different rules than the org, the repo should explicitly override:

```markdown
## Styling
This project uses Tailwind CSS (override: not CSS Modules).
```

### When Path-Specific Conflicts with Repo-Wide

```
Repo:   "All exports must be named exports"
Path:   "React components may use default exports for lazy loading"

→ Both are sent to Copilot
→ Path-specific is more specific context, so it typically wins
→ But this is NOT guaranteed
```

**Solution**: Make path-specific overrides explicit:

```markdown
---
applyTo: "src/pages/**/*.tsx"
---
# Page Component Rules
Exception to repo-wide rule: Page components use default exports
for Next.js dynamic import compatibility.
```

---

## Practical Guidelines

### Do

- ✅ Use organization level for truly universal rules (5-10 lines max)
- ✅ Use repository level for project-specific conventions
- ✅ Use path-specific for domain boundaries
- ✅ Make overrides explicit ("Override: ...")
- ✅ Test precedence by checking what Copilot actually produces

### Don't

- ❌ Put project-specific rules at organization level
- ❌ Rely on precedence to resolve conflicts — eliminate contradictions
- ❌ Have personal instructions that contradict team standards
- ❌ Assume Copilot will always pick the "right" instruction in a conflict

---

## Decision Matrix

| If the rule applies to... | Put it in... |
|---------------------------|-------------|
| Every repo in the org | Organization level |
| This entire repo | `copilot-instructions.md` |
| Only frontend code | `frontend.instructions.md` (applyTo: `src/components/**`) |
| Only my personal style | Personal settings |
| A specific workflow task | `.prompt.md` file (not an instruction file) |
