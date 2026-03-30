# Custom Review Agents

> Building your own AI code reviewer tailored to your organization's specific needs.

---

## When to Build Custom

| Use Case | Solution |
|----------|---------|
| Standard code review | Use Copilot/CodeRabbit (don't build) |
| Domain-specific rules | Custom instructions first, then custom agent |
| Proprietary analysis | Custom agent with org-specific knowledge |
| Integration with internal tools | Custom agent with API access |
| Regulatory compliance checking | Custom agent with compliance rules |

---

## Architecture: GitHub Actions Custom Reviewer

```yaml
name: Custom AI Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Get diff
        id: diff
        run: |
          DIFF=$(git diff origin/main...HEAD)
          echo "diff<<EOF" >> $GITHUB_OUTPUT
          echo "$DIFF" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
      
      - name: AI Review
        uses: actions/github-script@v7
        with:
          script: |
            const diff = `${{ steps.diff.outputs.diff }}`;
            
            // Call your AI API with the diff + your custom rules
            const response = await fetch('https://api.openai.com/v1/chat/completions', {
              method: 'POST',
              headers: {
                'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
                'Content-Type': 'application/json'
              },
              body: JSON.stringify({
                model: 'gpt-4o',
                messages: [{
                  role: 'system',
                  content: `You are a code reviewer for our fintech platform.
                    Check for: PCI compliance, proper error handling,
                    audit logging on financial transactions.`
                }, {
                  role: 'user', 
                  content: `Review this diff:\n${diff}`
                }]
              })
            });
            
            const review = await response.json();
            
            // Post as PR comment
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `## 🤖 Custom AI Review\n\n${review.choices[0].message.content}`
            });
```

---

## Custom Agent with MCP

Use MCP to give your AI reviewer access to internal tools:

```typescript
// Custom MCP server for code review
const reviewServer = new Server({
  name: "code-review-server",
  version: "1.0.0"
});

// Tool: Check internal style guide
reviewServer.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "check_style_guide") {
    const rules = await fetchStyleGuide(request.params.arguments.language);
    return { content: [{ type: "text", text: JSON.stringify(rules) }] };
  }
  
  if (request.params.name === "check_compliance") {
    const findings = await runComplianceCheck(request.params.arguments.code);
    return { content: [{ type: "text", text: JSON.stringify(findings) }] };
  }
});
```

---

## Key Takeaways

1. **Don't build what exists** — Use Copilot/CodeRabbit for standard review
2. **Custom instructions before custom agents** — Much less effort, often sufficient
3. **GitHub Actions is the integration point** — PR events trigger your agent
4. **MCP enables tool access** — Give AI reviewer access to internal systems
5. **Start simple** — Comment-based review, then upgrade to inline suggestions
