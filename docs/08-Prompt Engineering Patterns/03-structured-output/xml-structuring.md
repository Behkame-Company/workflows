# XML Structuring

> Using XML tags to organize prompts and outputs — Anthropic's recommended approach.

---

## Why XML Tags?

XML tags help models parse complex prompts unambiguously. They separate instructions from context, examples from input, and different sections of output.

```xml
<instructions>Review this code for security issues</instructions>

<context>
This is a Node.js Express API handling user authentication.
We use bcrypt for password hashing and JWT for tokens.
</context>

<code>
// ... the actual code ...
</code>
```

---

## Core Tag Patterns

### Separating Prompt Components

```xml
<role>You are a senior security engineer</role>

<instructions>
Review the code below for OWASP Top 10 vulnerabilities.
Rate each finding by severity.
</instructions>

<constraints>
- Only flag real vulnerabilities, not theoretical risks
- Provide a fix for each finding
</constraints>

<code>
{{user_code}}
</code>
```

### Wrapping Examples

```xml
<examples>
<example>
  <input>function add(a, b) { return a + b }</input>
  <output>function add(a: number, b: number): number { return a + b }</output>
</example>
<example>
  <input>const greet = (name) => `Hello ${name}`</input>
  <output>const greet = (name: string): string => `Hello ${name}`</output>
</example>
</examples>

Now convert this function:
<input>{{user_function}}</input>
```

### Structured Output

```xml
For each issue, respond using this format:

<finding>
  <severity>high</severity>
  <location>line 42</location>
  <issue>SQL injection via string concatenation</issue>
  <fix>Use parameterized queries: db.query('SELECT * FROM users WHERE id = $1', [userId])</fix>
</finding>
```

---

## Multi-Document Context

```xml
<documents>
<document index="1">
  <source>src/auth/login.ts</source>
  <content>
  // ... file content ...
  </content>
</document>
<document index="2">
  <source>src/auth/middleware.ts</source>
  <content>
  // ... file content ...
  </content>
</document>
</documents>

<query>How do these two files handle token expiration?</query>
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Use consistent tag names | `<code>` everywhere, not `<code>` + `<source>` + `<snippet>` |
| Nest logically | `<examples>` contains `<example>` contains `<input>` + `<output>` |
| Use descriptive names | `<instructions>` not `<i>`, `<constraints>` not `<c>` |
| Don't over-tag | Tag boundaries, not every sentence |
| Keep tags in prompts and output format | Consistency helps the model |

---

## XML vs Other Structuring

| Method | Best For | Model Support |
|--------|----------|--------------|
| XML tags | Claude (Anthropic) | ★★★ native support |
| Markdown headers | General use, documentation | ★★ all models |
| JSON structure | Data output, APIs | ★★ all models |
| YAML | Configuration, metadata | ★ workable |

Claude specifically recommends XML for prompt structuring. Other models handle it well too but may prefer markdown.

---

## Next Steps

- 🔗 [Schema Enforcement](schema-enforcement.md) — Constraining output to match schemas
- 🔗 [Markdown Output](markdown-output.md) — Generating structured documentation
