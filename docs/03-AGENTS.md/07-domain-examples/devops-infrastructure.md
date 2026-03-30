# DevOps & Infrastructure AGENTS.md Examples

> AGENTS.md for CI/CD pipelines, Docker, Terraform, Kubernetes, and infrastructure-as-code projects.

---

## Docker + Docker Compose Project

```markdown
# AGENTS.md

## Setup
- Build: `docker compose build`
- Up: `docker compose up -d`
- Down: `docker compose down`
- Logs: `docker compose logs -f [service]`
- Test: `docker compose run --rm app pnpm test`
- Shell: `docker compose exec app sh`

## Stack
Docker, Docker Compose, Node.js 20 (Alpine), PostgreSQL 16, Redis 7.

## Structure
docker/
├── Dockerfile           — Production multi-stage build
├── Dockerfile.dev       — Development with hot reload
└── docker-compose.yml   — Local development stack
scripts/                 — Build and deploy automation

## Conventions
- Multi-stage builds (builder → runner)
- Alpine base images for production
- Non-root user in production containers
- Health checks in all Dockerfiles
- .dockerignore always maintained alongside Dockerfile

## Docker Pattern
```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

# Stage 2: Run
FROM node:20-alpine AS runner
WORKDIR /app
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -S appuser
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
HEALTHCHECK CMD wget -q --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

## Boundaries
- NEVER use `latest` tag for base images (pin exact versions)
- NEVER run containers as root in production
- NEVER copy node_modules into Docker image (install in container)
- NEVER store secrets in Docker images or docker-compose.yml
- NEVER use `docker compose up` in production (use orchestrator)
```

---

## Terraform Infrastructure

```markdown
# AGENTS.md

## Setup
- Init: `terraform init`
- Plan: `terraform plan -var-file=environments/dev.tfvars`
- Apply: `terraform apply -var-file=environments/dev.tfvars`
- Validate: `terraform validate`
- Format: `terraform fmt -recursive`
- Lint: `tflint --recursive`

## Stack
Terraform 1.7, AWS provider 5.x, Terragrunt (optional).

## Structure
modules/              — Reusable Terraform modules
environments/
├── dev.tfvars        — Development variables
├── staging.tfvars    — Staging variables
└── prod.tfvars       — Production variables (NEVER modify without approval)
main.tf               — Root module
variables.tf          — Variable declarations
outputs.tf            — Output values
providers.tf          — Provider configuration
backend.tf            — State backend configuration

## Conventions
- Modules for all reusable infrastructure
- Variables for all configurable values (never hardcode)
- Tags on all resources: `environment`, `team`, `project`
- Outputs for all values needed by other modules
- Description on all variables and outputs

## Resource Pattern
```hcl
resource "aws_s3_bucket" "assets" {
  bucket = "${var.project}-${var.environment}-assets"

  tags = merge(var.common_tags, {
    Name = "${var.project}-assets"
  })
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

## Boundaries
- NEVER hardcode AWS account IDs or region
- NEVER store state locally (use S3 + DynamoDB backend)
- NEVER modify prod.tfvars without infrastructure team review
- NEVER use `terraform apply` without reviewing `terraform plan` output
- NEVER commit .terraform/ directory or .tfstate files
- NEVER destroy resources without explicit approval
```

---

## GitHub Actions CI/CD

```markdown
# AGENTS.md

## Setup
- CI runs on: push to main, PRs
- CD runs on: release tags (v*)
- Local test: `act -j build` (if act installed)

## Structure
.github/
├── workflows/
│   ├── ci.yml              — Build, test, lint on PRs
│   ├── cd.yml              — Deploy on release
│   ├── security.yml        — Dependency and code scanning
│   └── agents-md-check.yml — Validate AGENTS.md
├── actions/
│   └── setup/              — Composite action for env setup
└── CODEOWNERS

## Conventions
- Pin all action versions with SHA (never `@main` or `@v3`)
- Use composite actions for repeated setup steps
- Cache dependencies (node_modules, pip cache)
- Matrix strategy for multi-version testing
- Secrets stored in GitHub Secrets (never in workflow files)

## Workflow Pattern
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test
      - run: pnpm build
```

## Boundaries
- NEVER use `@main` or `@latest` for action versions
- NEVER store secrets in workflow files
- NEVER grant `write` permissions unless required
- NEVER use `pull_request_target` without understanding security implications
- NEVER commit workflow files without CI team review
```

---

## Kubernetes (Helm + Kustomize)

```markdown
# AGENTS.md

## Setup
- Lint charts: `helm lint charts/app`
- Template: `helm template app charts/app -f values/dev.yaml`
- Dry run: `helm upgrade --install app charts/app -f values/dev.yaml --dry-run`
- Kustomize build: `kustomize build overlays/dev`

## Structure
charts/
└── app/
    ├── Chart.yaml
    ├── values.yaml         — Default values
    ├── templates/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   ├── ingress.yaml
    │   └── _helpers.tpl
    └── tests/
values/
├── dev.yaml
├── staging.yaml
└── prod.yaml              — NEVER modify without SRE review

## Conventions
- All manifests templated via Helm (never raw YAML in production)
- Resource limits required on all containers
- Liveness and readiness probes on all deployments
- PodDisruptionBudget for all production workloads
- Secrets managed by External Secrets Operator (never in values files)

## Boundaries
- NEVER hardcode image tags (use Chart.appVersion or values)
- NEVER set replicas < 2 in production
- NEVER remove resource limits
- NEVER store secrets in values files
- NEVER modify prod.yaml without SRE approval
- NEVER use `kubectl apply` directly (use Helm/Kustomize)
```

---

## Key DevOps Anti-Patterns to Prevent

| Anti-Pattern | AGENTS.md Rule |
|-------------|----------------|
| Latest image tags | "NEVER use :latest — pin specific versions" |
| Root containers | "NEVER run as root in production" |
| Secrets in code | "NEVER store secrets in config files, env files, or manifests" |
| No health checks | "All containers must have health check endpoints" |
| Unpinned actions | "Pin all GitHub Actions with SHA, never @main" |
| Direct production changes | "NEVER modify production without review and approval" |
| Local state files | "NEVER store Terraform state locally" |
