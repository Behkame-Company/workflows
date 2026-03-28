# Troubleshooting Prompts

> When your prompts don't produce the results you need.

---

## Model Ignores My Instructions

**Symptom**: Model does something different from what you asked.

**Diagnoses**:
| Check | Fix |
|-------|-----|
| Instructions ambiguous? | Apply the "new employee" test — be more specific |
| Too many rules? | Reduce to 10-15 critical rules |
| Contradictory constraints? | Remove or prioritize conflicting rules |
| Instructions in the middle of long context? | Move to start or end |
| Using negative instructions? | Rewrite as positive: "Do X" not "Don't do Y" |

---

## Model Outputs Wrong Format

**Symptom**: JSON when you wanted prose, markdown when you wanted plain text, etc.

**Fixes**:
1. Add an explicit format specification: "Respond with JSON matching this schema:"
2. Add an example of the desired output
3. Match your prompt style to desired output (markdown-free prompt → less markdown output)
4. For JSON: "Respond with ONLY valid JSON. No markdown fencing, no explanation."

---

## Model Generates Incorrect Code

**Symptom**: Code has bugs, wrong patterns, or doesn't compile.

**Fixes**:
1. Reference an existing file: "Follow the pattern in src/api/users.ts"
2. Provide the interfaces/types the code needs to satisfy
3. Include test cases as specification
4. Use Chain-of-Thought: "Think through the edge cases before coding"
5. Add a self-check: "Verify your code handles null, empty, and too-large inputs"

---

## Model Gives Overly Generic Responses

**Symptom**: Textbook answers that don't match your project.

**Fixes**:
1. Add project context: stack, conventions, file structure
2. Reference specific project files
3. Use few-shot examples from your codebase
4. Be specific about the scope: "In our Next.js 14 App Router project..."

---

## Model Is Too Verbose

**Symptom**: Long explanations, unnecessary code, over-engineered solutions.

**Fixes**:
1. Set length limits: "Respond in 3 bullet points" or "Under 20 lines of code"
2. Be directive: "Give me only the code, no explanation"
3. Add constraint: "Implement the simplest solution that works"
4. For Claude 4.6: Lower the effort parameter

---

## Model Hallucinates Files or APIs

**Symptom**: References files that don't exist, APIs that don't exist.

**Fixes**:
1. "Read the file before referencing it — do not guess at contents"
2. "Use grep/glob to verify files exist before referencing them"
3. Include relevant file paths in the prompt
4. "If you're unsure whether a function/API exists, check the documentation or source code first"

---

## Model Loses Context Mid-Conversation

**Symptom**: Model forgets earlier decisions, repeats completed work.

**Fixes**:
1. Use compaction to summarize older turns (see Context Window Management docs)
2. Record decisions in persistent files (decisions.md, progress.txt)
3. Start a new conversation with a clean context
4. Re-inject critical decisions at the start of the next prompt

---

## Prompt Works Sometimes, Not Always

**Symptom**: Same prompt gives different quality results on different inputs.

**Fixes**:
1. Add more examples (few-shot) to stabilize behavior
2. Add explicit handling for the failing cases
3. Use self-consistency (run 3 times, pick majority)
4. Temperature 0 for maximum consistency
5. Test with diverse inputs and refine for failures

---

## Next Steps

- 🔗 [FAQ](faq.md) — Quick answers to common questions
- 🔗 [Iterative Refinement](../09-best-practices/iterative-refinement.md) — Systematic improvement
