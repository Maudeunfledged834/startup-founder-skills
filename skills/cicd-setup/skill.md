---
name: cicd-setup
trigger: When the user needs to set up or improve CI/CD pipelines — GitHub Actions, GitLab CI, deployment automation, or says "set up CI", "automate deployment", "add tests to pipeline", "fix my build".
related: [code-review, security-review]
reads: [startup-context]
---

# CI/CD Setup

## When to Use

- The user needs CI/CD from scratch or wants to improve an existing pipeline
- They need environment-specific deployment stages (staging, production)
- Their builds are slow and need optimization (caching, parallelism)
- They need secrets management for deployment credentials

## Context Required

From `startup-context`: tech stack, deployment target, team size. Also detect or ask:
- Language and framework (auto-detect from repo when possible)
- Deployment target (Vercel, AWS, GCP, Fly.io, etc.)
- CI/CD platform (default: GitHub Actions)
- Environments (dev, staging, production) and existing tests
- Secrets needed for build or deploy

## Workflow

1. **Detect tech stack** — Scan repo for package.json, requirements.txt, go.mod, Cargo.toml, etc. Note the test runner, linter, and build tool.
2. **Choose pipeline stages** — Lint, Test (with coverage), Build, Security scan, Deploy.
3. **Generate pipeline config** — Write CI config (default: `.github/workflows/ci.yml`) with caching, test matrix if needed, and environment-specific deploy jobs.
4. **Configure secrets** — List required secrets and how to add them. Never hardcode.
5. **Add deployment stages** — Staging (auto on `main` merge) and production (manual approval or tag-based).
6. **Optimize** — Dependency caching, parallel jobs, conditional steps, path filters.
7. **Deliver config and instructions** — Full config file and setup steps.

## Output Format

```markdown
# CI/CD Pipeline: [Project Name]
## Pipeline Overview — Mermaid flowchart showing stages
## Pipeline Configuration — Full YAML config file
## Secrets Required — table: name, where to get, how to add
## Setup Instructions — step-by-step to activate
## Optimization Notes — caching strategy, estimated build time
```

## Frameworks & Best Practices

### GitHub Actions Config Patterns

**Caching strategies by ecosystem:**

| Ecosystem | Cache Path | Cache Key |
|-----------|-----------|-----------|
| Node.js | `~/.npm` or `node_modules` | `hashFiles('**/package-lock.json')` |
| Python | `~/.cache/pip` | `hashFiles('**/requirements*.txt')` |
| Go | `~/go/pkg/mod` | `hashFiles('**/go.sum')` |
| Rust | `~/.cargo/registry`, `target/` | `hashFiles('**/Cargo.lock')` |
| Ruby | `vendor/bundle` | `hashFiles('**/Gemfile.lock')` |

### Pipeline Best Practices

- **Lint first** — fail early before expensive test runs
- **Parallel tests** — split unit and integration into separate jobs
- **Cache aggressively** — cuts 30-60% off build times
- **Pin action versions** — SHA hashes, not tags, for supply chain security
- **Set timeouts** (`timeout-minutes: 15`) and **concurrency** settings

### Environment Strategy

| Environment | Trigger | Approval | Purpose |
|-------------|---------|----------|---------|
| **CI** | Every push/PR | None | Run lint + tests |
| **Staging** | Merge to `main` | None (auto) | Integration testing, QA |
| **Production** | Git tag or manual | Required | Live users |

### Secrets Management

- Never commit secrets to the repository; use environment-level secrets
- Use OIDC for cloud auth instead of long-lived keys when possible
- Rotate secrets every 90 days minimum

### Security Scanning in CI

- **All projects:** `trivy` for container/dependency scanning, `semgrep` for SAST
- **Node.js:** `npm audit`; **Python:** `pip-audit`, `bandit`; **Go:** `govulncheck`

## Related Skills

- `code-review` — chain to review the CI config itself before committing
- `security-review` — chain to add or audit security scanning stages in the pipeline

## Examples

**Example prompt:** "Set up CI/CD for my Next.js app deployed on Vercel."

**Good output snippet:**
```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }
      - run: npm ci
      - run: npm run lint && npx tsc --noEmit
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }
      - run: npm ci && npm test -- --coverage
  security:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high
```

**Example prompt:** "My Python CI takes 8 minutes, how do I speed it up?"

**Good output snippet:**
```
Three fixes to cut to ~3 minutes: (1) Add pip caching keyed on
hashFiles('requirements*.txt'), (2) split unit/integration tests into
parallel jobs, (3) add path filters to skip CI on docs-only changes.
```
