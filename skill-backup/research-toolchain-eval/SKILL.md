---
name: research-toolchain-eval
description: >
  Use this skill to evaluate and compare developer toolchain options: build tools, bundlers,
  linters, formatters, test runners, CI/CD pipelines, containerization, IaC tools, and
  developer productivity tools. Trigger when the user asks "which build tool should we use?",
  "should we switch from X to Y bundler?", "evaluate CI/CD options for our stack",
  "compare Jest vs Vitest", "which linter setup is best for our team?", "Webpack vs Vite vs
  esbuild?", "Docker vs Podman?", "Terraform vs Pulumi vs CDK?", or any question about
  choosing or replacing tools in the development or deployment workflow.
---

# Research: Toolchain Evaluation

## Purpose
Choose the right toolchain that fits the team's stack, size, and workflow — not the trendiest one. Optimize for: build speed, DX, CI/CD integration, and maintenance burden.

## Toolchain categories

### Build & Bundling (frontend)
| Tool | Best for | Watch out for |
|------|----------|---------------|
| Vite | Modern ESM projects, fast HMR, Vue/React | Production bundle via Rollup (different from dev) |
| esbuild | Maximum speed, Go-based | No code splitting in library mode until recently |
| Webpack 5 | Complex enterprise setups, module federation | Config complexity, slower than modern tools |
| Rollup | Library publishing (not apps) | No dev server, app bundling is awkward |
| Turbopack | Next.js 13+ apps (Rust-based) | Early maturity, Next.js specific |
| Parcel 2 | Zero-config apps | Less community support than Vite/Webpack |

### Test runners
| Tool | Best for | Watch out for |
|------|----------|---------------|
| Vitest | Vite projects (same config, fast) | Newer, smaller plugin ecosystem than Jest |
| Jest | Universal, large ecosystem | Slower without SWC/Babel optimization |
| Playwright | E2E browser testing | Heavier setup, not unit testing |
| Cypress | E2E, component testing | Slower CI runs, Firefox support improving |
| pytest | Python (de facto standard) | N/A |

### Linting & formatting
| Tool | Language | Notes |
|------|----------|-------|
| ESLint + Prettier | JS/TS | ESLint for rules, Prettier for formatting |
| Biome | JS/TS | Rust-based, replaces both ESLint + Prettier, 10–50x faster |
| Ruff | Python | Replaces flake8 + isort + black, 100x faster |
| golangci-lint | Go | Aggregates multiple linters |
| Clippy | Rust | Built-in, no alternative needed |

### CI/CD
| Tool | Best for | Watch out for |
|------|----------|---------------|
| GitHub Actions | GitHub repos, large marketplace | Cost at scale, minutes consumption |
| GitLab CI | GitLab repos, self-hosted option | YAML complexity |
| CircleCI | Fast parallelism, Docker-native | Credit cost model, external dependency |
| Buildkite | Large-scale, self-hosted agents | Requires infrastructure to manage agents |
| ArgoCD | Kubernetes GitOps deployments | Kubernetes only |

### Infrastructure as Code
| Tool | Best for | Watch out for |
|------|----------|---------------|
| Terraform | Multi-cloud, large community | State management complexity, HCL language |
| Pulumi | Code-first (TS/Python/Go) | Smaller community than Terraform |
| AWS CDK | AWS-only shops, TypeScript-native | AWS lock-in |
| Ansible | Configuration management, VM setup | Not great for cloud resource provisioning |

## Evaluation criteria

For each toolchain tool, assess:

| Criterion | Questions |
|-----------|-----------|
| **Speed** | Build time, test execution time, lint time — measure on the actual project |
| **DX** | Error messages quality, IDE integration, config complexity |
| **CI integration** | Docker image size, caching support, parallelism |
| **Migration cost** | Config migration, plugin parity, team learning curve |
| **Maintenance** | Release frequency, breaking change history, community size |
| **Ecosystem compatibility** | Works with the project's existing stack (framework, TS version, Node version) |

## Benchmarking the toolchain

```bash
# Measure build time
time npm run build 2>&1 | tail -5

# Measure test run time
time npm test 2>&1 | tail -5

# Measure lint time
time npx eslint . 2>&1 | tail -3
time npx biome check . 2>&1 | tail -3

# Compare with new tool (after installing)
time npx vite build 2>&1 | tail -5
```

Always benchmark on the **actual project**, not a synthetic hello-world — tool performance varies significantly with project size.

## Migration path assessment

For each candidate tool replacing an existing one:

1. **Config migration**: Is there an automated migration tool? (e.g., `eslint-to-biome`)
2. **Plugin parity**: List the 5 most critical current plugins — does the new tool have equivalents?
3. **Incremental adoption**: Can both tools run in parallel during transition?
4. **CI/CD impact**: Does it require changes to CI pipeline? Estimate hours.
5. **Team training**: Is there a doc/tutorial gap? Estimate onboarding time.

## Output format

```markdown
### Toolchain Recommendation: <category>

**Winner**: <tool>
**Runner-up**: <tool> (for teams with <specific constraint>)

**Why <winner>**:
1. <Reason tied to project constraint>
2. <Reason with measured data>
3. <Reason about team/maintenance fit>

**Migration complexity**: Low / Medium / High
**Estimated migration effort**: X hours / X days

**Migration steps**:
1. ...
2. ...

**What we lose**: <honest trade-off — no migration is free>
```

## Constraints
- Always measure build/test time on the **actual project** before recommending a speed improvement
- A tool with 50x faster lint means nothing if its rule set doesn't cover the project's requirements
- Never recommend migrating to a tool that is pre-1.0 or less than 1 year old without flagging the stability risk
- Config migration tools are often incomplete — always test on a branch before committing
