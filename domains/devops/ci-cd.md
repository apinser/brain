# CI/CD conventions

## Platform
- GitLab CI/CD (`.gitlab-ci.yml`)
- GitHub is used only for external collaboration — not for CI/CD pipelines

## Core principle: Build Once, Deploy Everywhere
- Artifacts are built once and promoted across environments — never rebuilt per environment
- Artifact identity guaranteed via SHA256 checksum
- Environment-specific configuration is injected at deploy time, never baked into the artifact

## Pipeline triggers

| Event | Pipeline | Purpose |
|-------|----------|---------|
| MR opened / updated | Validation | Lint, type check, tests |
| Merge to `main` | Build | Produce and store versioned artifact |
| Tag `vX.Y.Z` or `vX.Y.Z-hotfix.N` | Release | Promote artifact to target environment |

## Environments
- Environments are project and client specific — not defined globally
- Always declared explicitly in the project's `.gitlab-ci.yml`
- Minimum expected: at least one non-production environment before production

## Secrets
- Never stored in git — not in `.env`, not in config files, not in CI yaml
- Always use GitLab CI/CD secret variables (masked + protected)
- Secrets scoped to the environment where they are needed

## Artifact promotion
- Artifacts stored in GitLab Package Registry after build
- Promotion from one environment to the next uses the same artifact (same SHA256)
- A release pipeline must verify the checksum before deploying

## Hotfix flow
1. Create `hotfix/description` branch from `main`
2. Apply fix — MR back to `main`
3. Tag the merge commit: `vX.Y.Z-hotfix.N`
4. Release pipeline triggers on the hotfix tag