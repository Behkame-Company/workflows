# Domain Guide: DevOps & CI/CD

> Instruction patterns for pipelines, Docker, infrastructure, and deployment.

---

## Sample: infrastructure.instructions.md

```yaml
---
applyTo: "infrastructure/**"
---
```

```markdown
# Infrastructure Instructions

## Docker
- Base images: use official slim variants (node:20-slim, python:3.12-slim)
- Multi-stage builds: separate build and runtime stages
- Never run containers as root — use `USER node` or equivalent
- No secrets in Dockerfiles — use build args or runtime env vars
- `.dockerignore` must exclude: node_modules, .git, .env, tests

## GitHub Actions
- Use pinned action versions: `actions/checkout@v4` not `@latest`
- Secrets via `${{ secrets.NAME }}`, never hardcode
- Cache dependencies: `actions/cache` for node_modules, pip cache
- Matrix builds for multiple Node/Python versions
- Fail fast on lint/type errors before running slower tests

## Terraform
- All resources must have descriptive names and tags
- Use modules for reusable infrastructure components
- State stored in remote backend (S3 + DynamoDB lock)
- Never store state files in the repository
- Plan before apply: always review `terraform plan` output

## Kubernetes
- Resource limits and requests on all pods
- Health checks (liveness + readiness probes) on all services
- Secrets via Kubernetes Secrets or external secrets operator
- Never use `latest` tag for container images — use specific SHAs or semver
```

---

## Sample: ci-config.instructions.md

```yaml
---
applyTo: ".github/workflows/**"
---
```

```markdown
# CI/CD Workflow Instructions

## GitHub Actions Best Practices
- Jobs should be independent where possible (parallelize)
- Use `needs:` for dependencies between jobs
- Cache aggressively: dependencies, build artifacts, Docker layers
- Timeout-minutes on all jobs (prevent runaway builds)
- Use environment protection rules for production deployments

## Workflow Structure
- CI: lint → type-check → test → build (fail fast)
- CD: CI passes → deploy to staging → smoke tests → deploy to production
- Separate workflows for: CI, deployment, dependency updates, scheduled tasks

## Do Not
- Never use `continue-on-error: true` without documentation
- Never store secrets in workflow files
- Never use self-hosted runners for untrusted input (fork PRs)
- Never skip CI checks for "quick fixes"
```

---

## Deployment Instructions

```yaml
---
applyTo: "scripts/deploy*"
---
```

```markdown
# Deployment Script Instructions
- All deployments must be idempotent (safe to run twice)
- Deployments log: who deployed, when, what version, which environment
- Rollback procedure must be documented in the script header
- Health check after deployment: verify /health endpoint returns 200
- Never deploy to production without passing staging first
```

---

## Key DevOps Conventions to Encode

| Convention | Why |
|-----------|-----|
| Pinned versions | Reproducible builds |
| No root containers | Security principle of least privilege |
| Cached dependencies | Faster CI runs |
| Fail-fast pipeline | Quick feedback on errors |
| Environment protection | Prevent accidental production deployments |
