# JSON Output Patterns

> Techniques for getting reliable, parseable JSON from language models.

---

## The Challenge

LLMs generate text, not data structures. Getting valid JSON requires explicit steering. Without it, you get:
- JSON wrapped in markdown code fences
- Missing closing braces
- Trailing commas (invalid JSON)
- Explanatory text mixed with JSON

---

## Technique 1: Explicit Format Specification

```
Analyze this code and respond with ONLY valid JSON (no markdown, no explanation):

{
  "issues": [
    {
      "severity": "high|medium|low",
      "line": <number>,
      "description": "<one sentence>",
      "fix": "<suggested code>"
    }
  ],
  "summary": "<one paragraph>"
}
```

**Key**: Show the exact schema. Include type hints for each field.

---

## Technique 2: JSON Schema Constraint

```
Respond with JSON matching this TypeScript interface:

interface AnalysisResult {
  bugs: Array<{
    file: string;
    line: number;
    severity: 'critical' | 'high' | 'medium' | 'low';
    description: string;
  }>;
  score: number; // 0-100
  recommendation: string;
}
```

TypeScript interfaces are an excellent way to specify JSON shape because models understand them well.

---

## Technique 3: API-Level Enforcement

### OpenAI Structured Output
```python
response = client.chat.completions.create(
    model="gpt-4",
    response_format={"type": "json_object"},
    messages=[...]
)
```

### Anthropic — Use Instruction + Schema
Anthropic doesn't have a JSON mode switch, but explicit schemas in the prompt work reliably:
```
Your response must be valid JSON matching this schema. 
No other text, no markdown fencing.
```

---

## Common JSON Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| JSON in markdown fences | Model wrapping in ` ```json ``` ` | "Respond with raw JSON, no markdown" |
| Trailing commas | Model mimicking JavaScript syntax | Validate and strip, or be explicit |
| Explanation before JSON | Model adding context | "Respond with ONLY the JSON object" |
| Truncated output | Response cut off by max_tokens | Increase max_tokens, reduce schema size |
| String escaping issues | Quotes in values | Provide examples with escaped strings |

---

## Validation Pipeline

Always validate JSON output programmatically:

```typescript
function parseAIResponse<T>(raw: string, schema: z.ZodSchema<T>): T {
  // Strip markdown fencing if present
  const cleaned = raw.replace(/```json\n?/g, '').replace(/```\n?/g, '').trim();
  
  const parsed = JSON.parse(cleaned);
  return schema.parse(parsed); // Zod validation
}
```

---

## Best Practices

1. **Always show the full schema** — Don't describe it, show it
2. **Use TypeScript interfaces or JSON Schema** — Models understand both well
3. **Add field descriptions** — `"severity": "critical|high|medium|low"` is clearer than `"severity": "string"`
4. **Include a complete example** — One valid JSON example prevents most format issues
5. **Validate on the receiving end** — Never trust raw model output as valid JSON
