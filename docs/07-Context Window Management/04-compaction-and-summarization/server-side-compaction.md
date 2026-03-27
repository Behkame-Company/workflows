# Server-Side Compaction

> Anthropic's automatic conversation summarization — the easiest way to manage long contexts.

---

## How It Works

Server-side compaction is a beta feature for Claude 4.6 models. When enabled:

1. **Detects** when input tokens exceed your trigger threshold
2. **Generates** a summary of the conversation up to that point
3. **Creates** a `compaction` block containing the summary
4. **Continues** the response with the compacted context

```
Request 1:  Normal conversation (under threshold)
Request 2:  Normal conversation (under threshold)
...
Request N:  Input exceeds 150K threshold
            → API generates summary (~5K tokens)
            → Creates compaction block
            → Continues response with compacted context
Request N+1: Only sends compaction block + recent turns
```

---

## Configuration

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8096,
    system="Your system prompt here",
    messages=messages,
    betas=["compact-2026-01-12"],
    context_management={
        "edits": [{
            "type": "compact_20260112",
            "trigger": {
                "type": "token_count",
                "token_count": 150000  # When to trigger (min: 50,000)
            },
            "pause_after_compaction": False,
            "instructions": None  # Custom summary prompt (optional)
        }]
    }
)
```

### Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `trigger.token_count` | 150,000 | When to trigger compaction |
| `pause_after_compaction` | `false` | Pause after summary for manual additions |
| `instructions` | Built-in | Custom summarization prompt |

---

## Passing Compaction Blocks

After compaction, pass the entire response back:

```python
# Append the full response (including compaction block) to messages
messages.append({"role": "assistant", "content": response.content})
messages.append({"role": "user", "content": "Continue with the next feature."})

# API automatically ignores content before the compaction block
next_response = client.messages.create(
    model="claude-sonnet-4-6",
    messages=messages,
    # ... same config
)
```

---

## Combining with Prompt Caching

```python
# Cache the system prompt separately
# When compaction occurs, system prompt cache stays valid
system_prompt = [{
    "type": "text",
    "text": "Your detailed system prompt...",
    "cache_control": {"type": "ephemeral"}  # Cache this
}]
```

This way, compaction only re-caches the summary, not the system prompt.

---

## Token Budget Enforcement

Use `pause_after_compaction` with a counter to enforce total budget:

```python
compaction_count = 0
MAX_COMPACTIONS = 5  # ~5 context windows worth of work

# When response has stop_reason "compaction":
compaction_count += 1
if compaction_count >= MAX_COMPACTIONS:
    # Inject "wrap up" instruction
    messages.append({
        "role": "user",
        "content": "Budget nearly exhausted. Complete current task and write progress notes."
    })
```

---

## Key Takeaways

1. **Easiest compaction** — Anthropic handles summarization automatically
2. **Configure trigger** — Set to 75% of your window (e.g., 150K for 200K)
3. **Pass blocks back** — Append full response including compaction block
4. **Cache system prompt** — Survives compaction, saves re-caching cost
5. **Budget enforcement** — Count compactions to limit total spend

---

## Next Steps

- 🔗 [Manual Summarization Strategies](manual-summarization-strategies.md) — DIY alternatives
- 🔗 [Compaction Tuning](compaction-tuning.md) — Optimizing what's preserved
