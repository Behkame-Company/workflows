# Prompts — Reusable Interaction Templates

> Prompts are pre-built, parameterized templates that guide AI behavior for specific tasks.

---

## What Are MCP Prompts?

Prompts are **server-provided templates** that structure how the AI approaches specific tasks. They're like recipes: "When the user wants X, use this structure."

Unlike tools (which execute) and resources (which provide data), prompts **shape the AI's reasoning**.

---

## Prompt Definition

```json
{
  "name": "code_review",
  "description": "Generate a structured code review for a pull request",
  "arguments": [
    {
      "name": "pr_number",
      "description": "The pull request number to review",
      "required": true
    },
    {
      "name": "focus_area",
      "description": "What to focus on: security, performance, style, or all",
      "required": false
    }
  ]
}
```

---

## Prompt Operations

### Discovery
```json
{ "method": "prompts/list", "id": 1 }

// Response
{
  "result": {
    "prompts": [
      {
        "name": "code_review",
        "description": "Structured code review for a PR"
      },
      {
        "name": "release_notes",
        "description": "Generate release notes from recent commits"
      }
    ]
  }
}
```

### Getting a Prompt
```json
{
  "method": "prompts/get",
  "id": 2,
  "params": {
    "name": "code_review",
    "arguments": { "pr_number": "247", "focus_area": "security" }
  }
}

// Response
{
  "result": {
    "messages": [
      {
        "role": "system",
        "content": {
          "type": "text",
          "text": "You are a senior security engineer reviewing PR #247. Focus on authentication, input validation, SQL injection, and XSS vulnerabilities. Structure your review as: Summary → Critical Issues → Warnings → Suggestions."
        }
      },
      {
        "role": "user",
        "content": {
          "type": "resource",
          "resource": {
            "uri": "github://pulls/247/diff",
            "mimeType": "text/x-diff"
          }
        }
      }
    ]
  }
}
```

---

## Prompts vs Tools vs Resources

| Aspect | Prompts | Tools | Resources |
|--------|---------|-------|-----------|
| **Purpose** | Shape AI behavior | Execute actions | Provide data |
| **Direction** | Guides the model | Model calls server | Model reads from server |
| **State change** | No | Yes (side effects) | No |
| **Output** | Messages for AI | Action results | Data content |
| **Analogy** | Recipe | Kitchen appliance | Ingredient |

---

## Practical Use Cases

### 1. Structured Code Review
Template that ensures consistent, thorough reviews across the team.

### 2. Release Notes Generation
Takes commit range, generates formatted release notes following team conventions.

### 3. Bug Report Template
Guides the AI to ask the right questions and format a complete bug report.

### 4. Migration Script Review
Specialized prompt for database migration safety checks.

### 5. Onboarding Walkthrough
Guides new developers through the codebase using resources and explanations.

---

## Design Best Practices

1. **Parameterize everything variable** — Don't hard-code values that change per invocation
2. **Include context via resources** — Combine prompts with embedded resource references
3. **Structure the output** — Tell the AI what format to produce
4. **Keep prompts focused** — One prompt per task, not one mega-prompt
5. **Version your prompts** — As team practices evolve, update prompt templates

---

## Key Takeaways

1. Prompts are **templates** that structure AI behavior for specific tasks
2. They're parameterized — arguments customize the template per invocation
3. They can **embed resources** to provide context alongside instructions
4. Use prompts for **repeatable workflows** that need consistency
5. They complement tools and resources — shaping *how* the AI uses them
