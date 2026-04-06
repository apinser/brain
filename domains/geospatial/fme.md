# FME conventions

## Version
- Always use the latest available FME version unless explicitly stated otherwise in the project
- Version pinning must be documented in the project ADR

## Products
- **FME Form**: local workspace development and testing
- **FME Flow**: production execution, scheduling, automation

## PythonCaller rules
- Keep PythonCallers minimal — input/output wiring and delegation only
- Any logic beyond trivial transformation must live in an external library imported at runtime
- One PythonCaller per logical operation — no monolithic scripts
- Always use the canonical startup script below as the basis for every PythonCaller

### Python version requirement
FME embeds its own Python interpreter. It must be configured to run Python ≥ 3.10.
Verify and document the FME Python version in the project before writing any PythonCaller.
All conventions from `general/python.md` apply without exception.

### Canonical startup script
Copy and adapt for every PythonCaller. Replace `"project.module"` with the appropriate logger name.

```python
import fme
import fmeobjects
import sys
from pathlib import Path
import logging

class _FMELogHandler(logging.Handler):
    _LEVEL_MAP = {
        logging.DEBUG:    fmeobjects.FME_INFORM,
        logging.INFO:     fmeobjects.FME_INFORM,
        logging.WARNING:  fmeobjects.FME_WARN,
        logging.ERROR:    fmeobjects.FME_ERROR,
        logging.CRITICAL: fmeobjects.FME_FATAL,
    }
    def emit(self, record):
        severity = self._LEVEL_MAP.get(record.levelno, fmeobjects.FME_INFORM)
        log = fmeobjects.FMELogFile()
        log.logMessageString(self.format(record), severity)

_handler = _FMELogHandler()
_handler.setFormatter(logging.Formatter("%(name)s — %(message)s"))
logger = logging.getLogger("project.module")  # adapt per project
logger.setLevel(logging.DEBUG)
logger.addHandler(_handler)

def _is_running_on_fme_flow() -> bool:
    flow_macro_names: tuple[str, ...] = (
        "FME_JOB_ID",
        "FME_SERVER_DEST_DIR",
        "FME_SHAREDRESOURCE_ENGINE",
    )
    for macro_name in flow_macro_names:
        logger.debug("%s : %s", macro_name, fme.macroValues.get(macro_name))
    return any(
        bool(fme.macroValues.get(macro_name))
        for macro_name in flow_macro_names
    )

def _add_local_package_path() -> None:
    workspace_dir: Path = Path(fme.macroValues["FME_MF_DIR"])
    repo_root: Path = workspace_dir.parent.parent
    shared_src_dir: Path = repo_root / "package"
    shared_src_dir_str: str = str(shared_src_dir.resolve())
    if not shared_src_dir.exists():
        raise RuntimeError(
            f"Local shared package path not found: {shared_src_dir_str}"
        )
    if shared_src_dir_str not in sys.path:
        sys.path.insert(0, shared_src_dir_str)
        logger.debug("Added to sys.path: %s", shared_src_dir_str)

if not _is_running_on_fme_flow():
    _add_local_package_path()
else:
    logger.debug("FME Flow execution — no sys.path modification")
```

## External library structure
When logic is extracted from a PythonCaller into a library:
- Place it under `package/` at the repo root
- Structure it as a proper Python package with `__init__.py`
- Follow all conventions from `general/python.md` (except Python version caveats above)
- The library must be importable from both FME Form (via sys.path injection) and FME Flow (via engine Python path)

## FME Flow
- Workspaces published to FME Flow must be tested locally in FME Form first
- Use FME Flow repositories to separate concerns (one repository per domain or project)
- Sensitive parameters (credentials, endpoints) via FME Flow Web Services or published parameters backed by environment variables — never hardcoded