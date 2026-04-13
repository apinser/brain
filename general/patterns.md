# Architecture patterns

## Decoupling
- Use `Protocol` to define interfaces — no inheritance required, statically verified by pyright
- Use `ABC` only when the base class carries shared behavior
- Dependencies flow inward: adapters depend on domain, never the reverse
- Inject dependencies via constructor or function parameter — no global state

## Layering
Three layers, strictly separated:

| Layer | Responsibility | May import |
|---|---|---|
| `domain` | Business logic, entities, rules | nothing project-internal |
| `service` | Orchestration, use cases | `domain` only |
| `adapter` | I/O, DB, API, files, CLI | `domain`, `service` |

A domain function never knows where data comes from or goes. An adapter never contains business logic.

## Configuration
- `pydantic-settings` for all configuration — validated at startup, fails fast on missing values
- Sources: environment variables first, `.env` file as local fallback
- Config object instantiated once at entry point, injected where needed
- No `os.environ.get()` scattered across the codebase

## Credentials
- Never in source code, never in git — not even in `.env` files committed to the repo
- `.env` always in `.gitignore`
- Use environment variables or a secret manager in production

## Logging
- `logging` stdlib only — never `print` in production code
- Log at boundaries: entry and exit of services and adapters
- Include context (IDs, operation names) — never log raw credentials or PII
- Level discipline: `DEBUG` for internals, `INFO` for operations, `WARNING` for recoverable issues, `ERROR` for failures

## Resilience
- Wrap external service calls (SFTP, REST API, Azure Blob, etc.) with retry + exponential backoff + jitter
- Parameters: `max_retries` (default 3), `backoff_factor` (default 2.0), base delay (default 1s)
- Jitter: add random noise to avoid thundering herd (`delay * (0.5 + random())`)
- Raise the original exception after all retries are exhausted — never swallow the final failure
- Implement as a decorator to keep call sites clean:
  ```python
  @retry_with_exponential_backoff(max_retries=3, backoff_factor=2.0)
  def upload(self, content: bytes, blob_name: str) -> str: ...
  ```
- Log each retry attempt at `WARNING` level with attempt number and delay

## Evolution
This file captures only patterns with proven transversal value.
Add a pattern here only when it has proven relevant across at least two projects.