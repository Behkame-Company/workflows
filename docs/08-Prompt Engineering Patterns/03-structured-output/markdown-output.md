# Markdown Output

> Generating well-structured documentation and reports from AI models.

---

## When to Use Markdown Output

- README files and documentation
- Code review reports
- Architecture Decision Records (ADRs)
- Meeting notes / summaries
- Changelog entries

---

## Controlling Markdown Format

### Specify Structure Explicitly

```
Write a README.md for this project with these sections:
1. # Project Name — one line description
2. ## Quick Start — 3-5 commands to get running
3. ## Architecture — bullet list of key components
4. ## API Reference — table of endpoints
5. ## Contributing — short guide

Keep the total under 200 lines.
```

### Control Heading Levels

```
Use these heading levels:
- # for the document title (one only)
- ## for major sections
- ### for subsections
- Do NOT use #### or deeper
```

### Reducing Excessive Markdown

Modern Claude models are less verbose, but if you're getting too much formatting:

```
Write in plain prose paragraphs. Do not use markdown headers, 
bullet lists, or code fences unless showing actual code. 
Write as if composing an email to a colleague.
```

Or match your prompt style to the desired output — prompts without markdown often produce responses without markdown.

---

## Table Generation

```
Create a comparison table with these columns:
| Feature | Library A | Library B | Library C |

Include rows for: performance, bundle size, TypeScript support, 
community size, and learning curve.
```

**Tip**: Providing the table header gets much more consistent formatting than describing the table in prose.

---

## Document Templates

### ADR Template
```
Generate an Architecture Decision Record using this structure:

# ADR-[number]: [Title]

## Status
[Proposed/Accepted/Deprecated/Superseded]

## Context
[What is the issue that we're seeing that is motivating this decision?]

## Decision
[What is the change that we're proposing and/or doing?]

## Consequences
[What becomes easier or more difficult to do because of this change?]
```

### Changelog Entry
```
Write a changelog entry for version 2.3.0 in Keep a Changelog format:

## [2.3.0] - YYYY-MM-DD
### Added
### Changed
### Fixed
### Removed
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Provide the heading structure | Model fills in content under your framework |
| Set a length limit | "Under 100 lines" prevents over-generation |
| Match prompt style to output style | Markdown-free prompt → less markdown in response |
| Show the template | Filling a template is more reliable than generating from description |
| Specify what to include AND exclude | "Include examples. Do not include installation for every OS." |

---

## Next Steps

- 🔗 [Templates](../11-practical/templates.md) — Ready-to-use prompt templates
- 🔗 [JSON Output Patterns](json-output-patterns.md) — When you need structured data instead
