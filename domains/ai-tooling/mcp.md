# MCP (Model Context Protocol) conventions

## Scope
MCP servers are the standard integration pattern between LLM agents and external tools/services.
No other LLM integration patterns (LangChain, AutoGen, direct API calls) are currently in use.

## Tool output format

### Principle: return only what the agent cannot infer itself
The agent already knows:
- Which tool it called
- What parameters it sent
- Whether the call succeeded (errors are exceptions, not JSON fields)

**Standard happy-path response:**
```json
{
  "results": [...],
  "count": 10
}
```
- `results` — always present, the actual payload
- `count` — include when the agent needs to reason about completeness or pagination
- Nothing else — no `tool`, no `timestamp`, no `parameters_used`, no `success`

**Errors — raise as MCP exceptions, never return as JSON:**
```python
raise McpError(ErrorCode.InternalError, "Descriptive message for the agent")
```

Error messages must be actionable — the agent must understand what went wrong and what to try next.

## Tool design
- One tool = one well-defined operation — no multi-purpose tools
- Tool names: `verb_noun` in snake_case (e.g. `search_locations`, `get_parcel_info`)
- Tool descriptions must be explicit enough for the agent to decide autonomously when to use them
- Required parameters only — no optional parameters that replicate defaults
- Validate inputs at tool entry — raise `McpError` immediately on invalid input

## FME Flow MCP pattern
FME Flow 2026.2+ exposes workspaces as MCP tools.

**Current implementation pattern** (Creator → PythonCaller → Writer):
```
Creator
  └─ PythonCaller (algorithm logic in Python)
        └─ Writer (JSON output)
```

Rules:
- Business logic lives in an external Python library — PythonCaller only orchestrates
- Output writer always produces JSON — follow the standard tool output format above
- One workspace = one MCP tool

**Evolution path**: once FME MCP tooling matures, migrate from PythonCaller-heavy workspaces
toward native FME transformers. Document the migration decision as an ADR.

## GitHub / GitLab workflow for AI-assisted development
- **GitHub** (`brain` + project repos): source of context for Claude — read and written during development sessions
- **GitLab**: authoritative project reference, CI/CD pipelines, client delivery
- At project close or major milestone: sync relevant conventions discovered during development back to `brain`
- GitLab repos are never connected to Claude directly — GitHub mirrors serve as the Claude-facing interface