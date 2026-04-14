# ADR-0001 — Require Ruff format before Python commits

**Date:** 2026-04-14
**Status:** Accepted

## Context
Python commits were being created before formatter execution, which left many post-commit formatting changes in working trees. This caused noisy diffs, reduced commit quality, and frequent follow-up cleanup work.

## Decision
We require `ruff format` to be run before any commit that includes Python files.

Mandatory order for Python changes:
1. `uv run ruff format .`
2. `git add ...`
3. `git commit ...`

Committing Python code before formatting is not allowed. If formatting introduces additional changes, those changes must be included in the same logical commit, or the commit must be redone.

## Consequences
What does this imply?
- **Positive:** cleaner history, fewer formatting-only leftovers, more predictable reviews.
- **Negative / tradeoffs accepted:** a small extra step before commit, and occasional larger commit payload due to auto-formatting.
- **What becomes harder:** quickly committing partial Python edits without running formatter first.