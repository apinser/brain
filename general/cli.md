# CLI conventions

## Command structure
- Prefer one command per intent.
- Avoid compound commands with `cd` and `git`; use separate calls.
- Avoid shell chaining that hides which step failed.
- Prefer explicit directory changes over inline directory switching.

## Readability
- Prefer commands that are easy to inspect and replay.
- Keep commands short and explicit when possible.
- When a workflow has multiple steps, run them as separate commands unless atomicity is required.

## Safety
- Avoid destructive commands unless explicitly validated.
- Prefer non-interactive commands in automated or agent-driven workflows.
- Validate the current working directory before running commands that modify repository state.

## Evolution
Add conventions here only when they apply across multiple projects and are not specific to a single tool.
