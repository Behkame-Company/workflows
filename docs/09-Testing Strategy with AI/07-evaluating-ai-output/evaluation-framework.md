# AI Output Evaluation Framework

> Designing systematic evaluation pipelines to measure AI-generated code quality — with task-specific criteria, automated grading, and high-volume testing.

---

## Overview

When AI generates code, tests, or documentation, you need a systematic way to evaluate quality. Ad-hoc review doesn't scale. This framework, drawing from Anthropic's eval best practices, establishes principles for designing evaluations that are task-specific, automated, and run at high volume.

## Evaluation Design Principles

### 1. Task-Specific

Every evaluation targets a specific capability. A generic "is this code good?" eval produces vague results. Instead:

| ❌ Generic | ✅ Task-Specific |
|-----------|-----------------|
| "Evaluate code quality" | "Does the function handle null inputs without throwing?" |
| "Are the tests good?" | "Do the tests cover all branches in the switch statement?" |
| "Is the output correct?" | "Does the API response match the OpenAPI schema?" |

### 2. Automated

Evaluations should run without human intervention. Manual review is too slow for iterating on prompts or comparing model versions:

```
Prompt Change → Generate 200 outputs → Auto-grade all → 
Compare scores → Decide if improvement
```

### 3. High Volume

A single example proves nothing. Run evaluations across many inputs to get statistical confidence:

- **Minimum viable eval set:** 20 cases
- **Confident eval set:** 50–100 cases
- **Production eval set:** 200+ cases with diverse inputs

## Success Criteria: SMAR

Define what "good" looks like before you start evaluating:

| Criterion | Definition | Example |
|-----------|-----------|---------|
| **Specific** | Precisely defined, no ambiguity | "Function returns ISO 8601 date string" |
| **Measurable** | Can be checked programmatically | `output.match(/^\d{4}-\d{2}-\d{2}T/)` |
| **Achievable** | Within the model's capability | Not "write a perfect compiler" |
| **Relevant** | Matches what users actually need | Test for real-world inputs, not contrived ones |

## Three Grading Methods

### Method 1: Code-Based Grading

Best for outputs with deterministic correct answers.

**Exact Match**
```python
def grade_exact(output: str, expected: str) -> bool:
    return output.strip() == expected.strip()
```

**String/Pattern Match**
```python
import re

def grade_pattern(output: str, pattern: str) -> bool:
    """Check if output matches expected pattern."""
    return bool(re.search(pattern, output))

# Example: Verify function signature is correct
grade_pattern(
    output=generated_code,
    pattern=r"def calculate_tax\(amount:\s*float,\s*rate:\s*float\)\s*->\s*float:"
)
```

**Custom Logic**
```python
import ast
import subprocess

def grade_code_output(generated_code: str, test_cases: list[dict]) -> dict:
    """Run generated code against test cases."""
    results = {"passed": 0, "failed": 0, "errors": []}
    
    for case in test_cases:
        try:
            # Write code to temp module and execute
            result = execute_function(generated_code, case["input"])
            if result == case["expected"]:
                results["passed"] += 1
            else:
                results["failed"] += 1
                results["errors"].append({
                    "input": case["input"],
                    "expected": case["expected"],
                    "actual": result
                })
        except Exception as e:
            results["failed"] += 1
            results["errors"].append({"input": case["input"], "error": str(e)})
    
    results["score"] = results["passed"] / len(test_cases)
    return results
```

**When to use:** Function outputs, API responses, data transformations, format compliance.

### Method 2: LLM-Based Grading

Best for subjective or complex outputs where code-based checks are insufficient.

**Key principle:** Ask the grader to **reason first, then score** (avoids anchoring bias).

```python
GRADING_PROMPT = """
Evaluate the following AI-generated test suite against the rubric.

## Code Under Test
{source_code}

## Generated Test Suite
{generated_tests}

## Rubric
- Coverage: Does it test all public methods? (0-3)
- Edge cases: Does it include boundary conditions? (0-3)
- Assertions: Are assertions specific and meaningful? (0-3)
- Independence: Can each test run in isolation? (0-3)
- Readability: Are test names descriptive? (0-3)

## Instructions
First, analyze each rubric dimension with specific observations.
Then provide your scores.
Format your response as:

REASONING:
[Your detailed analysis for each dimension]

SCORES:
Coverage: X/3
Edge cases: X/3
Assertions: X/3
Independence: X/3
Readability: X/3
Total: X/15
"""
```

**When to use:** Code review quality, documentation clarity, test comprehensiveness, architectural decisions.

### Method 3: Human Grading

Use as a last resort — for calibrating automated graders or evaluating novel outputs.

```python
# Human grading interface
{
    "task_id": "eval-042",
    "prompt": "Generate error handling for the payment service",
    "output": "...(generated code)...",
    "criteria": [
        "Does it handle network timeouts?",
        "Does it handle invalid card numbers?",
        "Does it log errors appropriately?",
        "Does it return user-friendly error messages?"
    ],
    "grader_notes": "",
    "scores": {
        "network_timeout": null,    # 0 or 1
        "invalid_card": null,        # 0 or 1
        "error_logging": null,       # 0 or 1
        "user_messages": null        # 0 or 1
    }
}
```

**When to use:** Initial calibration, novel evaluation criteria, dispute resolution, spot-checking automated graders.

## Choosing the Right Method

```
Is the output deterministic with a known correct answer?
├── YES → Code-based grading (exact match, pattern, custom logic)
└── NO
    ├── Can you write a rubric with clear criteria?
    │   ├── YES → LLM-based grading
    │   └── NO → Human grading (and develop a rubric from results)
    └── Is this a one-time evaluation?
        ├── YES → Human grading
        └── NO → Invest in LLM-based grading with calibration
```

## Building an Eval Pipeline

### Pipeline Architecture

```
┌───────────┐     ┌──────────────┐     ┌──────────────┐
│  Eval Set  │────▶│  AI Model    │────▶│  Grading     │
│  (inputs)  │     │  (generate)  │     │  (evaluate)  │
└───────────┘     └──────────────┘     └──────┬───────┘
                                              │
                                       ┌──────▼───────┐
                                       │  Results DB  │
                                       │  (track)     │
                                       └──────┬───────┘
                                              │
                                       ┌──────▼───────┐
                                       │  Dashboard   │
                                       │  (compare)   │
                                       └──────────────┘
```

### Eval Criteria Table

Define criteria for each evaluation task:

| Criterion | Method | Weight | Pass Threshold | Description |
|-----------|--------|--------|---------------|-------------|
| Correctness | Code-based | 40% | 100% | All test cases pass |
| Completeness | LLM-based | 25% | 80% | All requirements addressed |
| Style | LLM-based | 15% | 70% | Follows project conventions |
| Performance | Code-based | 10% | 90% | Meets latency/memory targets |
| Security | Code-based | 10% | 100% | No known vulnerability patterns |

### Grading Example: End-to-End

```python
# 1. Define the eval case
eval_case = {
    "id": "auth-001",
    "prompt": "Write a password validation function that requires: 8+ chars, "
              "1 uppercase, 1 lowercase, 1 number, 1 special character",
    "test_cases": [
        {"input": "Str0ng!Pass", "expected": True},
        {"input": "weak", "expected": False},
        {"input": "NoSpecial1", "expected": False},
        {"input": "no_upper_1!", "expected": False},
        {"input": "", "expected": False},
        {"input": "A1!aaaaa", "expected": True},
    ]
}

# 2. Generate output from AI
generated_code = ai_model.generate(eval_case["prompt"])

# 3. Code-based grading: Run test cases
code_results = grade_code_output(generated_code, eval_case["test_cases"])

# 4. LLM-based grading: Check code quality
quality_score = llm_grade(
    generated_code, 
    rubric="readable, efficient, no security issues, handles edge cases"
)

# 5. Combined score
final_score = (code_results["score"] * 0.7) + (quality_score * 0.3)
print(f"Auth-001: {final_score:.0%}")
```
