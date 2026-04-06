# Brain — Global Memory

## Purpose
This repository is the single source of truth for all transversal decisions, conventions, and tooling across Alexandre's projects.
It must be read at the start of every session involving any of his projects.

## Session initialization — required
1. Read `general/python.md`
2. Read `general/git.md`
3. Read `general/patterns.md`
4. Read `decisions/README.md` (ADR index)

If the session involves starting a new project, also read `playbooks/start-new-project.md`.

## Domain routing
Each project declares its active domains in its own `CLAUDE.md` under:
```
domains: [domain1, domain2, ...]
```

**If `domains` is missing or empty: stop and ask the user to declare them before proceeding.**

Available domains and their files:

| Domain | Files to read | Typical projects |
|---|---|---|
| `geospatial` | `domains/geospatial/fme.md`, `domains/geospatial/gis.md` | FME pipelines, GIS, spatial data |
| `data-engineering` | `domains/data-engineering/ms-fabric.md` | MS Fabric pipelines, Medallion architecture, ingestion, exposition |
| `devops` | `domains/devops/ci-cd.md` | GitLab/GitHub CI, Docker, deployment |
| `ai-tooling` | `domains/ai-tooling/mcp.md` | MCP servers, LLM agents, AI integrations |

## Precedence
When instructions conflict, apply this order (highest to lowest):
1. Instructions given in the current chat
2. `CLAUDE.md` of the active project repo
3. This file (`brain/CLAUDE.md`)

A project may explicitly override any rule from brain — but must document the deviation in its own ADR.

## Write rules
- Any structural decision → create an ADR in `decisions/` (format: `decisions/ADR-XXXX-title-in-kebab-case.md`)
- Update `decisions/README.md` after every new ADR
- Never modify an ADR with status `Accepted` — create a new one that supersedes it
- All changes via Pull Request — never commit directly to main

## Language
- All files in this repository: **English**
- Working language with the user: **French**