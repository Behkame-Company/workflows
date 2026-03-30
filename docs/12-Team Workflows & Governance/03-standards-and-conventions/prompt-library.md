# Prompt Library

> A shared library of tested prompts that ensure consistent quality, faster onboarding, and best practices encoded as reusable templates. Your team's collective AI experience, distilled into copy-paste recipes.

---

## Why a Prompt Library?

| Without Prompt Library | With Prompt Library |
|---|---|
| Each developer invents prompts from scratch | Proven prompts ready to use |
| Quality varies by developer experience | Consistent output quality |
| New developers start at zero | New developers start with team's best practices |
| Good prompts stay in one person's head | Knowledge is shared and versioned |
| No way to improve systematically | Track usage, refine over time |

---

## Structure

### Organization by Task Category

```
.github/prompts/
├── code-generation/
│   ├── create-api-endpoint.prompt.md
│   ├── create-service-class.prompt.md
│   └── create-data-model.prompt.md
├── testing/
│   ├── generate-unit-tests.prompt.md
│   ├── generate-integration-tests.prompt.md
│   └── generate-edge-case-tests.prompt.md
├── review/
│   ├── security-review.prompt.md
│   ├── performance-review.prompt.md
│   └── accessibility-review.prompt.md
├── refactoring/
│   ├── extract-function.prompt.md
│   ├── simplify-conditional.prompt.md
│   └── add-error-handling.prompt.md
├── debugging/
│   ├── diagnose-error.prompt.md
│   └── trace-data-flow.prompt.md
└── documentation/
    ├── generate-jsdoc.prompt.md
    └── write-readme-section.prompt.md
```

### Prompt Entry Format

Each prompt file follows this template:

```markdown
# [Prompt Name]

## Description
[What this prompt does and when to use it]

## Template
```
[The prompt template with {{placeholders}}]
```

## Example Usage
```
[A concrete example with placeholders filled in]
```

## Expected Output
[What good output looks like]

## When to Use
- [Scenario 1]
- [Scenario 2]

## Known Limitations
- [Limitation 1]
- [Limitation 2]

## Tags
[category], [language], [difficulty]
```

---

## Starter Prompt Library: 10 Essential Prompts

### 1. Create API Endpoint

**Description:** Generates a REST API endpoint following team patterns.

**Template:**
```
Create a {{HTTP_METHOD}} endpoint at {{ROUTE_PATH}} that {{DESCRIPTION}}.

Requirements:
- Follow the pattern in {{EXAMPLE_FILE}}
- Input validation using zod
- Error handling with appropriate HTTP status codes
- Include request/response TypeScript types
- Add JSDoc documentation
```

**Example:**
```
Create a POST endpoint at /api/users that creates a new user account.

Requirements:
- Follow the pattern in src/routes/products.ts
- Input validation using zod
- Error handling with appropriate HTTP status codes
- Include request/response TypeScript types
- Add JSDoc documentation
```

---

### 2. Generate Unit Tests

**Description:** Creates comprehensive unit tests for an existing function.

**Template:**
```
Generate unit tests for the {{FUNCTION_NAME}} function in {{FILE_PATH}}.

Include tests for:
- Happy path with valid inputs
- Null and undefined inputs
- Empty strings/arrays where applicable
- Boundary values
- Error conditions
- Return type verification

Use the testing pattern from {{TEST_EXAMPLE_FILE}}.
Test framework: {{FRAMEWORK}}
```

---

### 3. Security Review

**Description:** AI-assisted security review of a code file or function.

**Template:**
```
Review {{FILE_PATH}} for security vulnerabilities.

Check for:
- SQL/NoSQL injection
- XSS (cross-site scripting)
- Authentication/authorization bypass
- Insecure direct object references
- Sensitive data exposure
- Input validation gaps
- Race conditions
- Insecure dependencies

For each issue found, provide:
1. Location (line or function)
2. Severity (Critical/High/Medium/Low)
3. Description of the vulnerability
4. Recommended fix with code example
```

---

### 4. Add Error Handling

**Description:** Adds comprehensive error handling to existing code.

**Template:**
```
Add error handling to {{FILE_PATH}} following our project's error pattern.

Requirements:
- Use {{ERROR_TYPE}} for all errors (e.g., Result<T, AppError>)
- Handle: network failures, validation errors, not-found, unauthorized
- Log errors with appropriate severity levels
- Never expose internal details to API consumers
- Preserve stack traces for debugging
- Follow the pattern in {{ERROR_EXAMPLE_FILE}}
```

---

### 5. Write Documentation

**Description:** Generates documentation for functions, modules, or APIs.

**Template:**
```
Write documentation for {{TARGET}} in {{FILE_PATH}}.

Include:
- JSDoc/docstring for each public function
- Parameter descriptions with types
- Return value description
- Example usage
- Throws/error conditions
- Any important notes or caveats

Follow the documentation style in {{DOC_EXAMPLE_FILE}}.
```

---

### 6. Refactor for Readability

**Description:** Improves code readability without changing behavior.

**Template:**
```
Refactor {{FILE_PATH}} for improved readability.

Goals:
- Extract complex conditions into named functions
- Replace magic numbers with named constants
- Simplify nested logic (reduce nesting depth)
- Improve variable names for clarity
- Add comments only where logic is non-obvious

Constraints:
- Do NOT change external behavior
- Do NOT change function signatures
- Do NOT add or remove dependencies
- Preserve all existing tests
```

---

### 7. Create Data Model

**Description:** Generates a data model with validation schema.

**Template:**
```
Create a data model for {{ENTITY_NAME}} with these fields:
{{FIELD_LIST}}

Requirements:
- TypeScript interface/type
- Zod validation schema
- Database model (Prisma/TypeORM/etc.)
- Create and update DTOs
- Follow naming conventions in {{MODEL_EXAMPLE_FILE}}
```

---

### 8. Diagnose Error

**Description:** Systematic debugging assistance.

**Template:**
```
Help me diagnose this error:

Error message: {{ERROR_MESSAGE}}
File: {{FILE_PATH}}
Line: {{LINE_NUMBER}}
Context: {{WHAT_YOU_WERE_DOING}}

Stack trace:
{{STACK_TRACE}}

What I've already tried:
{{ATTEMPTS}}

Please:
1. Explain the most likely cause
2. Show me how to verify the cause
3. Provide a fix with code
4. Explain how to prevent this in the future
```

---

### 9. Generate Integration Test

**Description:** Creates integration tests for API endpoints.

**Template:**
```
Generate integration tests for the {{ENDPOINT}} endpoint in {{FILE_PATH}}.

Test scenarios:
- Successful request with valid data
- Validation errors (missing fields, invalid types)
- Authentication required (no token, expired token)
- Authorization (wrong role)
- Not found (invalid ID)
- Conflict (duplicate resource)
- Server error handling

Use the test setup pattern from {{TEST_SETUP_FILE}}.
Test framework: {{FRAMEWORK}}
```

---

### 10. Code Review Prompt

**Description:** AI-assisted pre-review for a code change.

**Template:**
```
Review the following code changes for potential issues:

{{CODE_DIFF_OR_FILE}}

Focus on:
1. Logic errors or incorrect assumptions
2. Security vulnerabilities
3. Missing error handling
4. Edge cases not covered
5. Performance concerns
6. Deviations from project patterns (see copilot-instructions.md)

For each issue:
- Severity: Critical / Major / Minor
- Location: specific line or section
- Problem: what's wrong
- Fix: recommended change
```

---

## Maintenance

### Contribution Process

```
1. Propose → Open a PR with the new prompt in .github/prompts/
2. Test   → Include at least 2 examples of the prompt being used
3. Review → Team reviews for quality, consistency, and usefulness
4. Merge  → Prompt is available to the whole team
```

### Regular Review

**Monthly:**
- Are all prompts still relevant?
- Any new patterns that should be templated?
- Any prompts producing poor results?

**Quarterly:**
- Full audit of the prompt library
- Deprecate outdated prompts
- Add prompts for new team patterns

### Deprecation

When a prompt is no longer useful:
```markdown
# ⚠️ DEPRECATED: [Prompt Name]
Deprecated on: 2025-07-15
Reason: Replaced by [new prompt] which handles [improvement]
Use instead: [link to replacement]
```

---

## Tracking Effectiveness

Create a simple feedback system:

```markdown
<!-- Add to the bottom of each prompt file -->
## Feedback
- 2025-07-10 @dev1: ⭐⭐⭐⭐⭐ "Saved 30 min on API endpoints"
- 2025-07-08 @dev2: ⭐⭐⭐ "Good for basic cases, needed tweaking for complex validation"
- 2025-06-28 @dev3: ⭐⭐⭐⭐ "Added edge case tests I wouldn't have thought of"
```
