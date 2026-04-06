# Git conventions

## Branching model

| Branch | Pattern | Lifetime | Purpose |
|--------|---------|----------|---------|
| `main` | `main` | Permanent | Production-ready, protected |
| Personal | `[initials]work` (e.g. `apwork`) | Long-lived | Primary working branch per developer |
| Feature | `feature/short-description` | Short-lived | Isolated feature when needed |
| Hotfix | `hotfix/short-description` | Short-lived | Urgent production fix |

Rules:
- `main` is protected — MR mandatory, no direct commits
- Personal branch is the default working branch for all developers
- Feature branches are optional — use when the work is large enough to isolate
- Hotfix branches are created from `main` and merged back to `main` via MR

## Commits
- Format: [Conventional Commits](https://www.conventionalcommits.org/)
- Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`
- Subject line: imperative, lowercase, no trailing period, max 72 chars
- Breaking changes: `feat!:` or `BREAKING CHANGE:` in footer
- One logical change per commit — no "fix everything" commits

## Merge requests
- MR mandatory to merge into `main` — no direct commits
- MR title follows Conventional Commits format
- At least one review before merge (self-review acceptable on solo projects)
- Squash merge preferred to keep `main` history linear

## Versioning and tags
- Semantic versioning: `MAJOR.MINOR.PATCH`
- Version bumps driven by commit types: `fix` → PATCH, `feat` → MINOR, `feat!` → MAJOR
- Standard release tag on `main`: `vX.Y.Z`
- Hotfix release tag on hotfix branch: `vX.Y.Z-hotfix.N`
- Tags are created only after MR is merged (except hotfix tags)

## Secrets
- Never committed to git — not in any branch, not in any file
- Use CI/CD secret variables or a secret manager