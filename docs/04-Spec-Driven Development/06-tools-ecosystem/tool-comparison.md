# SDD Tool Comparison Matrix

> Side-by-side comparison of all Spec-Driven Development tools.

---

## Feature Comparison

| Feature | Spec Kit | AWS Kiro | Tessl | DIY (Markdown) |
|---------|----------|----------|-------|----------------|
| **Cost** | Free (MIT) | Free preview | Paid | Free |
| **Setup** | CLI install | IDE download | Account signup | None |
| **Editor** | Any | Kiro IDE (Code OSS) | Any | Any |
| **Spec format** | Freeform MD | EARS format | Structured | Freeform MD |
| **Plan generation** | AI-assisted | AI-native | AI-assisted | Manual |
| **Task decomposition** | AI-assisted | AI-native | AI-assisted | Manual |
| **Implementation** | AI-assisted | AI-native | AI-orchestrated | Manual |
| **Quality gates** | Manual | Semi-automated | Automated | Manual |
| **Drift detection** | No | Via hooks | Yes | No |
| **Hooks/automation** | No | Yes | Yes | No |
| **Team collaboration** | Via Git | Built-in | Built-in | Via Git |
| **Version control** | Git-native | Git-native | Platform | Git-native |
| **AI model support** | Any | Bedrock | Multiple | Any |
| **Constitution/steering** | Yes | Yes (steering) | Yes | Manual |
| **Templates** | Customizable | Built-in | Built-in | Manual |

---

## AI Tool Compatibility

| SDD Tool | Copilot | Claude Code | Cursor | Codex | Gemini |
|----------|---------|-------------|--------|-------|--------|
| **Spec Kit** | ✅ Native | ✅ Via prompts | ✅ Via prompts | ✅ Via prompts | ✅ Via prompts |
| **AWS Kiro** | ❌ | ❌ | ❌ | ❌ | ❌ (Bedrock only) |
| **Tessl** | ✅ | ✅ | ✅ | ⚠️ | ✅ |
| **DIY** | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## Workflow Comparison

### Spec Kit Workflow
```
speckit init → speckit constitution → speckit specify →
speckit plan → speckit tasks → speckit implement
```
**Strengths**: Simple, tool-agnostic, customizable
**Weaknesses**: All manual quality gates, no automation

### Kiro Workflow
```
Open project → Describe feature → Review specs →
Review design → Review tasks → Implement with hooks
```
**Strengths**: Integrated, automated, hooks system
**Weaknesses**: IDE lock-in, model lock-in

### DIY Workflow
```
Create spec.md → Get AI to review → Create plan.md →
Get AI to create tasks.md → Implement task by task
```
**Strengths**: Maximum flexibility, zero dependencies
**Weaknesses**: No tooling, no guardrails, inconsistent

---

## Maturity Assessment

| Dimension | Spec Kit | Kiro | Tessl | DIY |
|-----------|----------|------|-------|-----|
| Documentation | ★★★★☆ | ★★★☆☆ | ★★☆☆☆ | N/A |
| Community | ★★★★☆ | ★★★☆☆ | ★★☆☆☆ | N/A |
| Enterprise readiness | ★★★☆☆ | ★★★★☆ | ★★★★☆ | ★☆☆☆☆ |
| Customizability | ★★★★★ | ★★★☆☆ | ★★★☆☆ | ★★★★★ |
| Learning curve | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★☆☆☆☆ |
| Active development | ★★★★★ | ★★★★★ | ★★★☆☆ | N/A |

---

## Recommendation by Team Profile

| Team Profile | Recommended Tool | Why |
|-------------|-----------------|-----|
| Solo developer, learning SDD | DIY | Start simple, learn the process |
| Small startup (2-5 devs) | Spec Kit | Lightweight, free, flexible |
| Web development team | Spec Kit + Copilot | Best integration with GitHub ecosystem |
| AWS-centric team | AWS Kiro | Native AWS integration |
| Enterprise (50+ devs) | Tessl or Kiro | Collaboration and governance features |
| Multi-tool team | Spec Kit | Tool-agnostic specs |
| Compliance-heavy industry | Kiro or Tessl | Audit trails and automation |

---

## Migration Paths

### From DIY to Spec Kit
```
1. Run `speckit init` in existing project
2. Move existing specs/ to .specify/specs/
3. Create constitution from existing standards
4. Start using CLI commands for new features
```

### From Spec Kit to Kiro
```
1. Open project in Kiro
2. Import .specify/ specs to Kiro format
3. Create steering files from constitution
4. Set up hooks for automation
```

---

*Next: [Integrating with AI Agents →](integrating-with-agents.md)*
