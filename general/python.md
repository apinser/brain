# Python conventions

## Version and runtime
- Python 3.10+ minimum
- Target: 3.12 on new projects
- Windows (dev) / Linux (CI and production)

## Typing
- Built-in generics: `list[str]`, `dict[str, int]`, `tuple[int, ...]`
- Union with `|`: `str | None`, never `Optional[str]`
- No `from __future__ import annotations` — target version supports it natively
- Annotate all function signatures and class attributes
- `TypeAlias` for complex reusable types
- `pyright` strict mode on new projects; not enforced retroactively on legacy code

## Code style
- Formatter: `ruff format`
- Linter: `ruff check`
- No `# type: ignore` without an explicit comment explaining why

## Structure and coupling
- Dependency inversion over direct imports between modules
- No circular imports — enforce via architecture, not tooling hacks
- One responsibility per module
- Prefer `pathlib.Path` over `os.path`
- Prefer `logging` over `print`

## Error handling
- Explicit exception types — never bare `except:`
- Fail fast: validate inputs at boundaries
- No silent failures — log or raise, never swallow

## Dependencies and environment
- Package manager: `uv`
- `pyproject.toml` for all projects
- Pin versions in production (`==`), use ranges in libraries (`>=`)
- Never install globally — always use `uv venv`

## What to challenge
If a proposed pattern deviates from the above, flag it and ask for explicit validation before proceeding.