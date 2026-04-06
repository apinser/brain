# Playbook: Start a new project

## 1. Create the GitHub repo
- Create a new repository on GitHub (public or private)
- Initialize with a `README.md`
- Default branch: `main` (protected — no direct commits)

## 2. Create the personal working branch
```bash
git checkout -b apwork
git push -u origin apwork
```

## 3. Add CLAUDE.md
Copy `CLAUDE-project-template.md` from `brain` and fill in:
- `domains:` — declare all active domains (mandatory — Claude will stop if missing)
- Project scope — one paragraph
- Project-specific constraints — or delete the section if none

## 4. Declare the project structure
Minimum expected structure:
```
project/
├── CLAUDE.md
├── README.md
├── docs/
│   └── decisions/       ← project-specific ADRs
└── src/
```
Add `pyproject.toml` and initialize `uv venv` if Python is involved.

## 5. Initialize the Python environment (if applicable)
```bash
uv venv
uv pip install -e ".[dev]"
```
`pyproject.toml` must define at minimum: project name, Python version constraint (≥ 3.10), dev dependencies (ruff, pyright).

## 6. Connect brain + project in Claude
In Claude.ai Projects:
- Add `brain` repo (read + write)
- Add the project repo (read + write)
- Select only the domain files relevant to the project

## 7. First session checklist
Verify Claude reads correctly:
- [ ] `brain/CLAUDE.md` initialization sequence followed
- [ ] Domain files loaded based on declared `domains:`
- [ ] Project `CLAUDE.md` scope understood
- [ ] `brain/decisions/README.md` consulted

## 8. GitLab mirror (when applicable)
When the project involves client delivery or CI/CD:
- Create the authoritative repo on GitLab
- GitHub repo remains the Claude-facing interface only
- Document the GitHub ↔ GitLab relationship in the project `README.md`