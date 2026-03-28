# Schema Enforcement

> Constraining model output to match a defined schema — from loose to strict.

---

## The Enforcement Spectrum

```
Loose                                              Strict
─────────────────────────────────────────────────────────►
"Return JSON"    Schema in      TypeScript      API-level
                 prompt         interface       enforcement
                 
 ~60% valid      ~85% valid     ~95% valid     ~99% valid
```

---

## Level 1: Verbal Request

```
Return your analysis as JSON.
```

**Reliability**: Low. Model may wrap in markdown, add explanations, or use wrong structure.

---

## Level 2: Schema in Prompt

```
Return JSON matching this exact structure:
{
  "status": "pass" | "fail",
  "issues": [{"line": number, "message": string}],
  "score": number
}
```

**Reliability**: Medium. Works for simple schemas.

---

## Level 3: TypeScript Interface

```
Respond with JSON matching this TypeScript type:

type ReviewResult = {
  status: 'pass' | 'fail' | 'warning';
  issues: Array<{
    file: string;
    line: number;
    severity: 'critical' | 'high' | 'medium' | 'low';
    message: string;
    suggestedFix?: string;
  }>;
  metrics: {
    totalIssues: number;
    criticalCount: number;
    score: number; // 0-100
  };
};
```

**Reliability**: High. Models understand TypeScript types well.

---

## Level 4: API-Level Enforcement

### OpenAI Structured Outputs
```python
from pydantic import BaseModel

class Issue(BaseModel):
    file: str
    line: int
    severity: str
    message: str

class ReviewResult(BaseModel):
    status: str
    issues: list[Issue]
    score: int

response = client.beta.chat.completions.parse(
    model="gpt-4o",
    response_format=ReviewResult,
    messages=[...]
)
```

### Anthropic (No built-in JSON mode)
Use strong prompting + client-side validation:
```python
import json
from pydantic import BaseModel, ValidationError

raw = response.content[0].text
try:
    data = ReviewResult.model_validate_json(raw)
except (json.JSONDecodeError, ValidationError) as e:
    # Retry with error feedback
    retry_prompt = f"Your previous response was invalid JSON: {e}. Try again."
```

---

## Retry Pattern for Schema Violations

```python
MAX_RETRIES = 3

for attempt in range(MAX_RETRIES):
    response = call_model(prompt)
    try:
        result = schema.parse(response)
        return result
    except ValidationError as e:
        prompt = f"""Your previous response didn't match the schema.
Error: {e}
Please try again, returning ONLY valid JSON matching the schema."""

raise SchemaError("Failed to get valid output after retries")
```

---

## Enum Enforcement

For fields with specific allowed values:

```
The "severity" field MUST be one of these exact strings:
- "critical" — System crash, data loss, security breach
- "high" — Feature broken, data corruption possible
- "medium" — Degraded functionality, workaround exists
- "low" — Cosmetic, minor inconvenience

Do not use any other values.
```

**Adding definitions** for each enum value dramatically improves selection accuracy.

---

## Best Practices

1. **Start with TypeScript interfaces** — They're the sweet spot of clarity and reliability
2. **Add field descriptions** — Especially for enums and ambiguous fields
3. **Show one complete example** — One valid JSON output prevents most format errors
4. **Validate programmatically** — Never trust raw output; always parse and validate
5. **Implement retry with error feedback** — Tell the model what went wrong
6. **Keep schemas small** — Large schemas increase error rate; split if needed

---

## Next Steps

- 🔗 [JSON Output Patterns](json-output-patterns.md) — Core JSON techniques
- 🔗 [Markdown Output](markdown-output.md) — When you need documentation format
