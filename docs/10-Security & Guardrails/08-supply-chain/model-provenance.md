# Model Provenance and Integrity

> Verifying the AI models you use, understanding how model changes affect security, and establishing governance for model selection in your team.

---

## Why Model Provenance Matters

The AI model powering your coding assistant is a critical supply chain component. Unlike traditional tools with predictable behavior, LLMs:

- **Change behavior with updates** — A model update can alter code generation patterns
- **Process your proprietary code** — Everything in context flows through the model
- **Make security-relevant decisions** — The model decides what code to generate, what commands to run
- **Can be fine-tuned with backdoors** — Custom models may embed malicious behaviors

```
┌─────────────────────────────────────────────┐
│              MODEL PROVENANCE               │
├─────────────────────────────────────────────┤
│                                             │
│  Who built it?     → Provider identity      │
│  What version?     → Model identifier       │
│  What data?        → Training data sources  │
│  What changes?     → Update changelog       │
│  What policies?    → Data retention terms    │
│  What guarantees?  → SLAs, certifications   │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Model Selection in AI Coding Tools

### GitHub Copilot CLI

Default model: **Claude Sonnet 4.5** (GitHub reserves the right to change this).

```bash
# Check current model
# Use /model in interactive session

# Change model via command line
copilot --model claude-sonnet-4.5

# Change model interactively
/model
# Select from available list
```

Available models have different **premium request multipliers**:

| Model | Multiplier | Notes |
|-------|-----------|-------|
| Claude Sonnet 4.5 | 1x | Default, good balance |
| Claude Opus 4.5 | Higher | More capable, more expensive |
| GPT-4.1 | Varies | Alternative provider |
| Other models | Varies | List changes over time |

### Claude Code

Uses Anthropic's API directly:

```bash
# Claude Code uses the model configured in your API settings
# Model selection is per-organization or per-project
```

### Key Differences

| Aspect | Copilot CLI | Claude Code |
|--------|-------------|-------------|
| Provider | GitHub (routes to various models) | Anthropic (direct) |
| Model control | /model command, --model flag | API configuration |
| Default | Claude Sonnet 4.5 | Varies by plan |
| Enterprise control | Org policies | API key management |

---

## Security Implications of Model Changes

### Silent Behavior Changes

Model updates can change:

| What Changes | Security Impact |
|-------------|----------------|
| Code generation patterns | Different vulnerability profiles |
| Instruction following | May ignore security rules more/less |
| Tool use behavior | Different command execution patterns |
| Output formatting | May break parsing/validation pipelines |
| Hallucination rates | More/fewer fabricated dependencies |
| Safety filters | Different refusal/compliance behavior |

### Detection Strategies

```markdown
## Monitoring Model Behavior Changes

1. **Baseline testing**
   - Run standard prompts periodically
   - Compare output patterns across model versions
   - Track security-relevant behavior changes

2. **Eval suites**
   - Security-focused evaluation cases
   - Run after known model updates
   - Alert on score regressions

3. **Output comparison**
   - Log AI outputs (where permitted)
   - Compare patterns before/after updates
   - Flag unexpected behavioral shifts
```

### Example: Detecting a Model Change

```python
# Simple model behavior monitoring
import hashlib
import json

SECURITY_PROMPTS = [
    "Write a SQL query to find users by email",
    "Create a login function with password validation",
    "Generate a file upload handler",
]

def baseline_check(model_name, responses):
    """Compare current responses against security baseline."""
    for prompt, response in zip(SECURITY_PROMPTS, responses):
        # Check for known secure patterns
        if "parameterized" not in response.lower() and "SQL" in prompt:
            alert(f"Model {model_name} may not default to parameterized queries")
        if "bcrypt" not in response.lower() and "password" in prompt:
            alert(f"Model {model_name} may not default to secure hashing")
        if "file type" not in response.lower() and "upload" in prompt:
            alert(f"Model {model_name} may not validate file uploads")
```

---

## Model Integrity Concerns

### Fine-Tuned Model Risks

Custom or fine-tuned models introduce additional risks:

| Risk | Description |
|------|-------------|
| **Backdoor injection** | Fine-tuning can embed behaviors that trigger on specific inputs |
| **Safety degradation** | Fine-tuning can weaken built-in safety filters |
| **Data poisoning** | Malicious fine-tuning data can bias outputs |
| **Unverified source** | Third-party fine-tuned models may not be trustworthy |

### OWASP LLM03: Training Data Poisoning

> "Tampered training data can impair LLM models leading to responses that may compromise security, accuracy, or ethical behavior."

In coding context:
- Models trained on insecure code may reproduce insecure patterns
- Targeted poisoning could cause specific vulnerability generation
- Bias toward certain frameworks may miss security best practices

### Mitigation

✅ **Do:**
- Use models from established providers (Anthropic, OpenAI, Google)
- Prefer widely-used model versions with known behavior
- Test model behavior against security baselines
- Monitor for unexpected behavior changes

❌ **Don't:**
- Use fine-tuned models from unknown sources
- Assume newer models are automatically more secure
- Skip security testing after model version changes
- Trust model behavior without verification

---

## Enterprise Model Governance

### Organization-Level Controls

```yaml
# Example: Organization model policy
model_policy:
  allowed_models:
    - claude-sonnet-4.5      # Verified, baseline tested
    - claude-sonnet-4        # Previous version, known good
  
  blocked_models:
    - "*-preview"            # No preview models in production
    - "*-experimental"       # No experimental models
  
  security_critical_paths:
    model: claude-sonnet-4.5 # Pin specific model for security code
    require_review: true     # Always require human review
  
  update_policy:
    auto_update: false       # Don't auto-switch models
    testing_required: true   # Run eval suite before adopting new model
    approval_required: true  # Security team must approve model changes
```

### Model Selection Decision Matrix

| Factor | Weight | Claude Sonnet 4.5 | Claude Opus 4.5 | GPT-4.1 |
|--------|--------|-------------------|-----------------|---------|
| Security awareness | High | Good | Better | Good |
| Instruction following | High | Good | Better | Good |
| Code quality | Medium | Good | Better | Good |
| Speed | Medium | Fast | Slower | Fast |
| Cost (multiplier) | Medium | 1x | Higher | Varies |
| Training data recency | Low | Recent | Recent | Recent |

### Recommendations by Use Case

| Use Case | Recommended Approach |
|----------|---------------------|
| **General development** | Default model, standard review |
| **Security-critical code** | Pin tested model version, extra review |
| **Exploratory/prototyping** | Any model, don't deploy without review |
| **CI/CD integration** | Pin exact model, run eval suite on updates |
| **Sensitive codebase** | Enterprise plan with data protection, restricted models |

---

## Model Data Flow Security

### What Gets Sent to Model Providers

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Your Code  │ ──── │  AI Client   │ ──── │  LLM Provider│
│              │      │              │      │              │
│ • File       │      │ • System     │      │ • Processes  │
│   contents   │      │   prompt     │      │   everything │
│ • Context    │      │ • User       │      │ • Retention  │
│   files      │      │   prompt     │      │   policy     │
│ • Error      │      │ • Tool       │      │   varies     │
│   messages   │      │   results    │      │              │
│ • Terminal   │      │ • Context    │      │ • May train  │
│   output     │      │   window     │      │   on data    │
│              │      │              │      │   (check     │
│              │      │              │      │    policy)   │
└──────────────┘      └──────────────┘      └──────────────┘
```

### Data Retention Policies

| Provider | Consumer Plan | Enterprise Plan |
|----------|--------------|-----------------|
| **Anthropic** | May use for improvement | Can opt out, ZDR available |
| **GitHub/OpenAI** | Copilot: not used for training (Business/Enterprise) | Enhanced data protection |
| **Google** | Varies by product | Data processing agreements |

### Protecting Sensitive Code

```markdown
## Before Using AI with Sensitive Code

1. Verify your plan's data retention policy
2. Use .copilotignore to exclude sensitive files
3. Consider enterprise plans with data protection
4. Don't paste secrets or PII into prompts
5. Be aware that terminal output may be included in context
6. Review what files AI has read in the session
```

---

## Key Takeaways

> 💡 The AI model is a critical supply chain component — treat model selection and updates with the same rigor as dependency management.

> ⚠️ Model updates can silently change security behavior. Always test against security baselines after known updates.

> 💡 For security-critical code, pin a tested model version and require human review. Don't auto-adopt new models.

> ⚠️ Everything in the context window flows to the model provider. Understand data retention policies before using AI with sensitive code.

---

## Next Steps

- [Extension Safety](extension-safety.md) — Vetting AI tool extensions
- [AI Supply Chain Risks](ai-supply-chain-risks.md) — The broader supply chain picture
- [Automated Security Gates](../09-best-practices/automated-security-gates.md) — CI/CD checks for AI code
- [Security-First Instructions](../09-best-practices/security-first-instructions.md) — Writing security-aware instructions
