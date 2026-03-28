# Custom Agent Design

> Designing specialized agents for your team — create focused, expert agents that follow your guidelines and handle specific tasks with consistency and quality.

---

## What Are Custom Agents?

Custom agents are specialized versions of AI coding agents configured for specific roles. Instead of a general-purpose assistant, you create an **expert** — a frontend engineer who knows your design system, a security reviewer who checks your specific threat model, a database specialist who follows your migration procedures.

```
┌──────────────────────────────────────────────────────────────┐
│            General Agent vs Custom Agents                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   General Agent                Custom Agents                 │
│   ─────────────                ─────────────                 │
│   Knows everything             Knows your project deeply     │
│   broadly                      in a focused area             │
│                                                              │
│   "AI Assistant"               "Frontend Expert"             │
│                                "API Designer"                │
│                                "Security Reviewer"           │
│                                "DB Specialist"               │
│                                "Doc Writer"                  │
│                                                              │
│   Jack of all trades           Master of one                 │
│   Follow general best          Follow YOUR team's            │
│   practices                    specific practices            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Built-In Specialized Agents

Copilot CLI includes built-in specialized agents for common tasks:

- **explore** — fast codebase exploration and question answering
- **task** — command execution with clean output (tests, builds, lints)
- **code-review** — high-signal code review (only real issues, never style nits)
- **general-purpose** — full-capability agent for complex multi-step tasks

These built-in agents demonstrate the principle: **focused agents outperform general agents** on their specialty.

## Custom Agent Design Process

```
┌──────────────────────────────────────────────────────────────┐
│             Agent Design Process                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   1. DEFINE ROLE                                             │
│      What is this agent an expert in?                        │
│      What does it do better than a general agent?            │
│           │                                                  │
│           ▼                                                  │
│   2. WRITE INSTRUCTIONS                                      │
│      System prompt with role, knowledge, constraints         │
│      Project-specific conventions and patterns               │
│           │                                                  │
│           ▼                                                  │
│   3. CONFIGURE TOOLS                                         │
│      Which tools does this agent need?                       │
│      Which tools should it NOT have?                         │
│           │                                                  │
│           ▼                                                  │
│   4. SET BOUNDARIES                                          │
│      What should it refuse to do?                            │
│      When should it ask for help?                            │
│           │                                                  │
│           ▼                                                  │
│   5. TEST EXTENSIVELY                                        │
│      Real tasks, edge cases, failure modes                   │
│      Does it follow instructions consistently?               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Step 1: Define the Role

Be specific about what the agent is and what it is not:

```markdown
## Agent: Frontend Expert

### Is
- Expert in React, TypeScript, and Tailwind CSS
- Knows our design system (components in src/components/ui/)
- Follows our accessibility standards (WCAG 2.1 AA)
- Creates responsive layouts mobile-first

### Is Not
- A backend developer (should not modify API routes)
- A database engineer (should not write migrations)
- A DevOps engineer (should not modify CI/CD)
```

## Step 2: Write System Instructions

The system instructions define the agent's personality, knowledge, and behavior:

```markdown
# Frontend Expert Agent

You are a senior frontend engineer specializing in React and TypeScript.
You build UI components for the Acme application.

## Your Knowledge
- Design system: src/components/ui/ (Button, Input, Card, Modal, etc.)
- Page layouts: src/components/layout/ (Header, Sidebar, Content)
- Feature components: src/features/[name]/components/
- Styling: Tailwind CSS with custom theme in tailwind.config.ts
- State: Zustand stores in src/stores/
- API: React Query hooks in src/hooks/api/

## Your Approach
1. Check if a UI component already exists before creating new ones
2. Use the design system — extend existing components, don't reinvent
3. Build mobile-first, test at 375px, 768px, and 1280px breakpoints
4. Always include loading, error, and empty states
5. Write tests with React Testing Library (behavior, not implementation)

## Your Standards
- TypeScript strict mode — no `any`, explicit return types on exports
- All interactive elements must be keyboard accessible
- All images must have alt text
- Color contrast minimum 4.5:1 for text
- Components must work without JavaScript (progressive enhancement)

## Your Process
1. Read the requirement
2. Check existing components for reuse
3. Create/modify component with TypeScript types
4. Add Storybook story
5. Write tests
6. Verify: `npm test`, `npx tsc --noEmit`, `npm run storybook -- --smoke-test`
```

## Step 3: Configure Available Tools

Different agents need different tools:

```
┌──────────────────────────────────────────────────────────────┐
│            Tool Configuration by Agent Role                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Frontend Expert       Security Reviewer     Doc Writer      │
│  ────────────────      ─────────────────     ──────────      │
│  ✓ File read/write     ✓ File read           ✓ File read     │
│  ✓ Shell (build/test)  ✓ Grep/search         ✓ File write    │
│  ✓ Git operations      ✓ Shell (read-only)   ✓ Grep/search   │
│  ✓ Browser preview     ✗ File write          ✓ Shell (docs)  │
│  ✗ Database access     ✗ Git write           ✗ Database      │
│  ✗ Deploy tools        ✗ Deploy              ✗ Deploy        │
│                                                              │
│  Principle: Least privilege — only tools the role needs      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Step 4: Set Boundaries and Guardrails

Define what the agent should escalate to a human:

```markdown
## Boundaries

### Always Do
- Run tests after every change
- Follow the design system
- Include TypeScript types
- Add accessibility attributes

### Never Do
- Modify backend code (src/api/, src/services/)
- Change database schemas
- Modify CI/CD configuration
- Delete test files
- Use !important in CSS

### Escalate When
- Requirement contradicts existing design system
- Change would affect more than 5 files
- Performance concern with current approach
- Security question about user input handling
```

## Step 5: Test the Agent

Test with real tasks your team actually handles:

```markdown
## Test Cases for Frontend Expert Agent

### Test 1: Simple Component
"Create a UserAvatar component that shows initials when no image is available"
Expected: Uses existing Avatar base, handles fallback, includes tests

### Test 2: Feature Implementation
"Add a search filter to the products page"
Expected: Uses existing Input, debounced, loading state, no results state

### Test 3: Boundary Test
"Update the database schema for the users table"
Expected: Agent refuses, explains this is outside its role

### Test 4: Edge Case
"Make the dashboard work on mobile"
Expected: Responsive design, no horizontal scroll, touch-friendly targets
```

## Agent Roles for Coding Teams

### Common Agent Configurations

```
┌────────────────────────────────────────────────────────┐
│  Role              Focus Area         Key Skills        │
├────────────────────────────────────────────────────────┤
│  Frontend Expert   UI Components      React, CSS,      │
│                    User Experience     Accessibility     │
│                                                        │
│  API Designer      Endpoint Design    REST, Schemas,   │
│                    Contract-First     Validation        │
│                                                        │
│  DB Specialist     Schema Design      Migrations,      │
│                    Query Optimization  Indexing          │
│                                                        │
│  Security          Vulnerability      OWASP, Auth,     │
│  Reviewer          Detection          Input Validation  │
│                                                        │
│  Documentation     API Docs           OpenAPI, README, │
│  Writer            User Guides        Code Comments     │
│                                                        │
│  Test Engineer     Test Strategy      Unit, E2E,       │
│                    Coverage Gaps      Edge Cases        │
│                                                        │
└────────────────────────────────────────────────────────┘
```

## Agent Design Template

Use this template when creating a new custom agent:

```markdown
# Agent: [Role Name]

## Identity
You are a [seniority level] [role] specializing in [focus area].
Your job is to [primary responsibility] for the [project name] project.

## Knowledge Base
- [Key technology 1]: [location in codebase]
- [Key technology 2]: [location in codebase]
- [Pattern/convention]: [description]
- [Pattern/convention]: [description]

## Process
1. [First step in standard workflow]
2. [Second step]
3. [Third step]
4. [Verification step with exact command]

## Quality Standards
- [Standard 1 with measurable criteria]
- [Standard 2 with measurable criteria]
- [Standard 3 with measurable criteria]

## Tool Access
- Allowed: [list of tools this agent uses]
- Denied: [list of tools this agent must not use]

## Boundaries
- Never: [hard constraint 1]
- Never: [hard constraint 2]
- Escalate: [condition that requires human input]
- Escalate: [condition that requires human input]

## Verification
After every task, run:
- [Verification command 1]
- [Verification command 2]
- [Verification command 3]
```

## Next Steps

- [Agent Composition](./agent-composition.md) — combine agents into powerful workflows
- [Building Custom Skills](./building-skills.md) — create skills that agents use
- [Agent Skills Specification](./skills-spec.md) — the standard for portable agent capabilities
- [Testing Agentic Systems](../09-best-practices/testing-agents.md) — evaluate your custom agents
