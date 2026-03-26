# Other SDD Tools

> Tessl, intent-driven.dev, and emerging alternatives.

---

## Tessl

### Overview
[Tessl](https://tessl.io) is a spec-driven development platform focused on enterprise teams using AI agents at scale.

### Key Features
- Collaborative spec editing (multi-user)
- Agent orchestration for implementation
- Spec analytics and quality scoring
- Enterprise access controls and audit trails

### Best For
Teams that need more structure than Spec Kit but don't want to switch IDEs (unlike Kiro).

---

## Intent-Driven Development (intent-driven.dev)

### Overview
[intent-driven.dev](https://intent-driven.dev) promotes a philosophy that sits between vibe coding and formal SDD — capturing developer *intent* as the primary artifact.

### Key Concepts
- **Intent documents** — lighter than specs, heavier than prompts
- **Progressive elaboration** — start with high-level intent, add detail as needed
- **Knowledge base** — accumulated best practices for intent-driven workflows

### Best For
Teams transitioning from vibe coding to SDD who want a gradual adoption path.

---

## Augment Code

### Overview
[Augment Code](https://www.augmentcode.com) provides guides and tooling for AI-assisted development workflows, including spec-driven approaches.

### Key Offerings
- Comprehensive SDD implementation guides
- Workflow templates for prompted AI development
- Comparison frameworks (SDD vs vibe coding)

---

## DIY Spec-Driven Workflows

You don't need a specialized tool to practice SDD. A "DIY" approach using standard tools:

### Minimal Setup
```
specs/                           # Regular Markdown files
├── constitution.md              # Project standards
├── active/
│   └── feature-name/
│       ├── spec.md              # Specification
│       ├── plan.md              # Technical plan
│       └── tasks.md             # Task breakdown
└── templates/
    └── feature-template.md      # Reusable template
```

### With GitHub Copilot CLI
```bash
# Create spec using Copilot
copilot "Help me write a spec for [feature]. 
Ask me clarifying questions before writing."

# Generate plan from spec
copilot "Read specs/active/notifications/spec.md and create a technical plan"

# Break into tasks
copilot "Read the plan and create an ordered task list"

# Implement
copilot "Implement Task 1 from specs/active/notifications/tasks.md"
```

### With Claude Code
```bash
claude "Read specs/active/notifications/spec.md.
Create a technical plan in plan.md that addresses every requirement."
```

---

## Emerging Tools and Approaches

| Tool / Approach | Status | Description |
|----------------|--------|-------------|
| **GitHub Spec Kit** | Production | Open-source SDD toolkit |
| **AWS Kiro** | GA | Spec-driven IDE |
| **Tessl** | Growing | Enterprise SDD platform |
| **intent-driven.dev** | Community | Intent-driven development methodology |
| **GSD (Get Shit Done)** | Active | Workflow framework with spec-driven phases |
| **Cursor Rules + Specs** | Community | Using .cursorrules with spec-like workflows |
| **Claude Projects + CLAUDE.md** | Active | Combining project context with spec files |

### The Trend
The industry is converging on spec-driven approaches. Every major AI coding tool is adding support for structured specifications. The specific tool matters less than the practice.

---

## Choosing the Right Tool

### Decision Matrix

| If You... | Consider... |
|-----------|-------------|
| Use GitHub Copilot | Spec Kit + copilot-instructions.md |
| Want a full IDE experience | AWS Kiro |
| Need enterprise features | Tessl |
| Want maximum flexibility | DIY with Markdown |
| Are just starting with SDD | intent-driven.dev philosophy + Spec Kit |
| Use multiple AI tools | Spec Kit (tool-agnostic) |

---

*Next: [Tool Comparison Matrix →](tool-comparison.md)*
