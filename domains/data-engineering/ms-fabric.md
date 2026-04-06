# MS Fabric — Data Engineering conventions

## Architecture
- Medallion pattern: **Bronze** (raw ingestion) → **Silver** (curated, exposed) → Gold (aggregated, optional)
- All data exposition goes through **Silver exclusively** — never directly from Bronze
- Exposition format (Excel, API, etc.) and destination (SharePoint, etc.) are project-specific

## Notebook configuration
- External config files stored in notebook `builtin/` resources
- Access path via `notebookutils.nbResPath`:
  ```python
  CONFIG_PATH: str = f"{notebookutils.nbResPath}/builtin/config.json"
  ```
- Config validated at startup — raise `RuntimeError` immediately on invalid config, never proceed silently

## Credentials
- All credentials via Azure Key Vault exclusively:
  ```python
  secret = mssparkutils.credentials.getSecret(KEY_VAULT_URI, "secret-name")
  ```
- Never hardcode credentials or connection strings in notebooks

## Logger
- Use `logging` stdlib — never a custom Logger class, never `print` in production code
- Fabric pre-configures the root logger with `MSPySparkLoggerHandler`
- Never call `logging.basicConfig(force=True)` — it removes the Fabric handler

Canonical logger setup (copy in every notebook):
```python
import logging
from notebookutils import mssparkutils

logger = logging.getLogger(mssparkutils.runtime.context["currentNotebookName"])
logger.setLevel(logging.DEBUG)
```

## Spark session
Standard configuration applied in every ingestion notebook:
```python
spark = SparkSession.builder.appName(SPARK_SESSION_NAME).getOrCreate()
spark.conf.set("spark.sql.shuffle.partitions", "24")
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "52428800")  # 50 MB
spark.conf.set("spark.databricks.delta.mergeSchemaInWrite", "false")
spark.conf.set("spark.databricks.delta.optimizeWrite.enabled", "true")
```

## Audit table
Every ingestion pipeline tracks execution via an `AuditTable` with the following status codes:

| Code | Meaning |
|------|---------|
| `-5` | Rollback |
| `-2` | Uncaptured exception |
| `-1` | Error |
| `0`  | In progress |
| `1`  | Success |

## Audit columns
Every ingested table carries mandatory audit columns:

| Column | Always present | Condition |
|--------|---------------|-----------|
| `audit_row_hash` | ✅ | — |
| `audit_job_id` | ✅ | — |
| `audit_loaded_at` | ✅ | — |
| `audit_is_deleted` | ❌ | Only if `"track_deletes": true` in source config |

## Ingestion sources
Supported source patterns (one `*IngestionConfig` class per source type, JSON-configured):
- SFTP + CSV (`CSVIngestionConfig`)
- SFTP + XML (`XMLIngestionConfig`)
- REST API + JSON (`RestApiJsonIngestionConfig`)

Each config class must implement a `validate()` method — called at startup before any data movement.

## Data exposition — GraphQL API
Access levels on Silver lakehouse views (`api` schema):

| Level | Audience | Data sensitivity |
|-------|----------|-----------------|
| `Enterprise` | Internal only | All data including sensitive |
| `Partners` | External partners | No sensitive data |
| `Open` | Any authenticated user | Public data only |

Views must be created in the `api` schema of Silver lakehouse.
Currently only `Enterprise` is in production.