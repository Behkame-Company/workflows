# LLM-Based Grading

> Using LLMs to evaluate LLM output — rubric design, reasoning-first scoring, and the LLM-as-judge pattern for scalable quality assessment.

---

## Overview

Some AI outputs can't be graded with simple code checks. When you need to evaluate code quality, documentation clarity, or test comprehensiveness, you can use an LLM as an automated grader. This meta-evaluation approach scales far better than human review — but requires careful rubric design to produce consistent, useful scores.

## The Core Principle: Reason First, Then Score

Anthropic's key recommendation for LLM-based grading: **ask the model to reason before scoring.** When the model assigns a score first, its reasoning conforms to justify that score (anchoring bias). When it reasons first, the score follows from the analysis.

### ❌ Score-First (Anchoring Bias)

```
Rate this code 1-5 for quality, then explain why.

Score: 4
Reasoning: The code is generally good because... [reasoning shaped to justify 4]
```

### ✅ Reason-First (Unbiased)

```
Analyze this code against each quality dimension below. 
Provide your detailed observations FIRST, then assign a score.

ANALYSIS:
- Error handling: The function catches TypeError but not ValueError...
- Edge cases: No handling for empty input...
- Naming: Variables are descriptive except 'x' on line 12...

SCORE: 2/5
```

The reasoning-first approach produces more accurate and consistent evaluations because the score emerges from the analysis rather than the other way around.

## Rubric Design

A rubric is the grading instruction you give to the LLM judge. Poor rubrics produce inconsistent scores. Good rubrics produce reliable, actionable evaluations.

### Rubric Qualities

| Quality | Bad Example | Good Example |
|---------|------------|--------------|
| **Specific** | "Is the code good?" | "Does the function validate all input parameters before processing?" |
| **Observable** | "Is it well-designed?" | "Does it separate I/O from business logic?" |
| **Scaled** | "Good or bad" | "0: No validation, 1: Partial, 2: All params validated with clear errors" |
| **Anchored** | "1-5 scale" | "3 = handles the stated requirements; 5 = handles requirements + edge cases + errors" |

### Rubric Template

```markdown
## Evaluation Rubric: [Task Name]

### Dimension 1: Correctness (0-3)
- **0 — Incorrect:** Output does not solve the stated problem
- **1 — Partially correct:** Solves the main case but fails on edge cases
- **2 — Mostly correct:** Handles main case and most edge cases
- **3 — Fully correct:** Handles all cases including boundaries and errors

### Dimension 2: Completeness (0-3)
- **0 — Incomplete:** Major requirements missing
- **1 — Partial:** Some requirements addressed, others missing
- **2 — Mostly complete:** All major requirements, minor gaps
- **3 — Complete:** All requirements fully addressed

### Dimension 3: Code Quality (0-3)
- **0 — Poor:** Unreadable, no error handling, magic numbers
- **1 — Below average:** Some issues with naming or structure
- **2 — Good:** Clean, readable, follows conventions
- **3 — Excellent:** Clean, well-documented, idiomatic

### Dimension 4: Security (0-3)
- **0 — Vulnerable:** Contains known vulnerability patterns
- **1 — Risky:** Missing input validation or sanitization
- **2 — Adequate:** Basic security practices followed
- **3 — Strong:** Comprehensive input validation, no injection risks

### Grading Instructions
1. Read the code carefully
2. Analyze each dimension with SPECIFIC observations (quote line numbers)
3. After ALL analysis is complete, assign scores
4. Provide an overall assessment and improvement suggestions
```

## The LLM-as-Judge Pattern

Send three components to the grading model:

```
┌─────────────────────────────────┐
│         Grading Prompt          │
├─────────────────────────────────┤
│  1. THE OUTPUT (code to grade)  │
│  2. THE RUBRIC (scoring guide)  │
│  3. REFERENCE (expected answer) │
│                                 │
│  → Reason about each dimension  │
│  → Then assign scores           │
│  → Then give overall verdict    │
└─────────────────────────────────┘
```

### Implementation

```python
def llm_grade(output: str, rubric: str, reference: str = None) -> dict:
    """Use an LLM to grade another LLM's output."""
    
    prompt = f"""You are an expert code reviewer grading AI-generated code.

## Output to Evaluate
```
{output}
```

## Grading Rubric
{rubric}
"""
    
    if reference:
        prompt += f"""
## Reference Solution
```
{reference}
```
Compare the output against this reference solution.
"""
    
    prompt += """
## Instructions
1. Analyze the output against EACH rubric dimension
2. Cite specific examples from the code (with line numbers)
3. Note both strengths and weaknesses
4. After completing all analysis, provide scores for each dimension
5. Calculate the total score

Format your response as:

ANALYSIS:
[Detailed dimension-by-dimension analysis]

SCORES:
Dimension 1: X/3 — [one-line justification]
Dimension 2: X/3 — [one-line justification]
Dimension 3: X/3 — [one-line justification]
Dimension 4: X/3 — [one-line justification]
Total: X/12

VERDICT: [PASS/FAIL] — [Summary and key improvement suggestions]
"""
    
    response = model.generate(prompt)
    return parse_grading_response(response)
```

### Parsing Grading Output

```python
import re

def parse_grading_response(response: str) -> dict:
    """Extract structured scores from LLM grading response."""
    scores = {}
    
    # Extract individual dimension scores
    score_pattern = r"Dimension \d+: (\d+)/(\d+)"
    matches = re.findall(score_pattern, response)
    for i, (score, max_score) in enumerate(matches):
        scores[f"dimension_{i+1}"] = {
            "score": int(score),
            "max": int(max_score)
        }
    
    # Extract total
    total_match = re.search(r"Total: (\d+)/(\d+)", response)
    if total_match:
        scores["total"] = {
            "score": int(total_match.group(1)),
            "max": int(total_match.group(2)),
            "percentage": int(total_match.group(1)) / int(total_match.group(2))
        }
    
    # Extract verdict
    verdict_match = re.search(r"VERDICT: (PASS|FAIL)", response)
    scores["verdict"] = verdict_match.group(1) if verdict_match else "UNKNOWN"
    scores["raw_response"] = response
    
    return scores
```

## Practical Example: Grading Test Quality

```python
RUBRIC = """
### Dimension 1: Coverage (0-3)
- 0: Tests only the happy path
- 1: Tests happy path + 1-2 error cases
- 2: Tests happy path + most error cases + some edge cases
- 3: Comprehensive coverage including boundaries, nulls, and concurrency

### Dimension 2: Assertion Quality (0-3)
- 0: No assertions or only checks truthiness
- 1: Basic equality checks
- 2: Specific value checks with meaningful error messages
- 3: Checks values, types, side effects, and error messages

### Dimension 3: Test Independence (0-3)
- 0: Tests share mutable state and fail when run in isolation
- 1: Some shared state but mostly independent
- 2: Independent but share setup logic
- 3: Fully independent with proper setup/teardown

### Dimension 4: Readability (0-3)
- 0: No descriptions, unclear what's being tested
- 1: Basic descriptions, some clarity
- 2: Clear descriptions following "should X when Y" pattern
- 3: Self-documenting with clear arrange-act-assert sections
"""

# Grade AI-generated tests
result = llm_grade(
    output=generated_test_code,
    rubric=RUBRIC,
    reference=reference_test_code
)

print(f"Score: {result['total']['percentage']:.0%}")
print(f"Verdict: {result['verdict']}")
```

## Limitations of LLM-Based Grading

### Known Biases

| Bias | Description | Mitigation |
|------|------------|------------|
| **Verbosity bias** | LLMs rate longer, more detailed outputs higher | Include "conciseness" in rubric; penalize unnecessary verbosity |
| **Self-preference** | Models may rate outputs from the same model family higher | Use a different model for grading than generation |
| **Position bias** | In comparisons, first or last items may be favored | Randomize ordering; run comparisons in both directions |
| **Anchoring** | Score-first responses anchor the reasoning | Always require reasoning before scoring |
| **Sycophancy** | Grader may be reluctant to give low scores | Explicitly state that low scores are expected and appropriate |

### Calibration Techniques

**1. Human Calibration Set**

Grade 20 outputs manually, then compare with LLM grades:

```python
human_scores = [3, 1, 2, 3, 2, 1, 0, 3, 2, 1, ...]
llm_scores   = [3, 2, 2, 3, 2, 1, 1, 3, 2, 2, ...]

# Calculate agreement
agreement = sum(h == l for h, l in zip(human_scores, llm_scores)) / len(human_scores)
print(f"Agreement: {agreement:.0%}")  # Target: >80%
```

**2. Spot-Check Outliers**

When the LLM grades an output much higher or lower than expected, manually review to calibrate:

```python
# Flag scores that deviate significantly from the mean
mean_score = sum(scores) / len(scores)
outliers = [s for s in scores if abs(s - mean_score) > 2 * std_dev]
```

## When to Use LLM Grading vs. Code-Based Grading

```
Use CODE-BASED grading when:
├── Output has a deterministic correct answer
├── You can write assertions (equality, pattern, type checks)
├── Speed and cost matter (code grading is free and instant)
└── You need 100% consistent results

Use LLM-BASED grading when:
├── Output quality is subjective (readability, design quality)
├── You can't easily codify the criteria
├── You need nuanced analysis with explanations
└── You're evaluating complex artifacts (tests, docs, architecture)

Use BOTH when:
├── You want correctness (code) AND quality (LLM) scores
├── Code checks verify the minimum, LLM checks evaluate the ceiling
└── You need a composite score across multiple dimensions
```
