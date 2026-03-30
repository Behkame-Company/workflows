# Prompt Chaining

> Sequential multi-step prompt workflows — breaking complex tasks into a pipeline.

---

## What Is Prompt Chaining?

Prompt chaining splits a complex task into sequential steps, where each step's output feeds into the next step's input. Each step is a separate prompt (and potentially a separate API call).

```
Step 1          Step 2          Step 3          Step 4
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Extract   │──►│ Analyze  │──►│ Generate │──►│ Validate │
│ context   │    │ patterns │    │ solution │    │ output   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

---

## Why Chain Instead of One Big Prompt?

| Single Prompt | Chained Prompts |
|--------------|----------------|
| All-or-nothing | Inspect intermediate results |
| Hard to debug | Each step testable independently |
| Complex, long prompt | Simple, focused prompts |
| Single failure point | Can retry individual steps |
| One model for everything | Can use different models per step |

---

## Common Chaining Patterns

### 1. Generate → Review → Refine

```
Step 1: "Generate a caching layer for our API"
  ↓ output: initial implementation
Step 2: "Review this code for thread safety and memory leaks"
  ↓ output: list of issues
Step 3: "Fix these issues in the implementation: [issues from step 2]"
  ↓ output: refined implementation
```

This is the most practical pattern — Anthropic calls it the "self-correction" pattern.

### 2. Extract → Transform → Load

```
Step 1: "Extract all API endpoint definitions from these files"
  ↓ output: list of endpoints with routes, methods, params
Step 2: "Generate OpenAPI 3.0 spec from these endpoint definitions"
  ↓ output: OpenAPI YAML
Step 3: "Generate TypeScript SDK client from this OpenAPI spec"
  ↓ output: TypeScript client code
```

### 3. Plan → Execute → Verify

```
Step 1: "Create a step-by-step plan to migrate from REST to GraphQL"
  ↓ output: migration plan
Step 2: "Execute step 1 of the plan: [plan step 1]"
  ↓ output: code changes for step 1
Step 3: "Verify: do these changes maintain backward compatibility?"
  ↓ output: compatibility report
```

---

## Implementation

### Manual Chaining (Copilot CLI)
Execute each prompt in sequence, copying the relevant output as context for the next.

### Programmatic Chaining
```python
def generate_and_review(task: str) -> str:
    # Step 1: Generate
    draft = call_model(f"Generate: {task}")
    
    # Step 2: Review
    issues = call_model(f"Review this code for bugs:\n{draft}")
    
    # Step 3: Refine (only if issues found)
    if "no issues" not in issues.lower():
        final = call_model(f"Fix these issues:\n{issues}\n\nOriginal:\n{draft}")
        return final
    
    return draft
```

---

## When to Chain vs Single Prompt

| Situation | Approach |
|-----------|----------|
| Simple task | Single prompt |
| Need to inspect intermediate results | Chain |
| Different expertise needed per step | Chain |
| Task has clear sequential stages | Chain |
| Steps are independent | Parallel prompts, not chain |
