---
name: fabric-notebook
description: Generate a well-structured Microsoft Fabric PySpark notebook. Use when writing data ingestion, transformation, semantic model refresh, or utility notebooks in the Fabric environment. Produces a complete notebook with standard structure, Fabric-specific imports, logging, and error handling baked in.
---

# Skill: /fabric-notebook

You are generating a Microsoft Fabric PySpark notebook. Every notebook you produce must follow the structure and conventions below exactly. Output is a ready-to-use `.ipynb`-compatible code structure — present each cell clearly labelled so the user can paste it into Fabric Notebook.

---

## Environment Reference — Fill In Per Project

Before generating a notebook, confirm these with the user (or read them from an existing project config) rather than inventing values:

| Item | Value |
|---|---|
| DEV workspace (groupId) | `<your-dev-workspace-guid>` |
| DEV dataset (datasetId) | `<your-dev-dataset-guid>` |
| PRD workspace (groupId) | `<your-prd-workspace-guid>` |
| PRD dataset (datasetId) | `<your-prd-dataset-guid>` |
| Auth pattern | Service Principal via a secrets vault (Key Vault or equivalent) |

---

## Standard Notebook Structure — Always Use These 5 Cells

### Cell 1 — Notebook Header (Markdown)
```markdown
# [Notebook Name]
**Purpose:** [One sentence — what this notebook does]
**Environment:** Microsoft Fabric (DEV / PRD)
**Author:** [Your name]
**Last updated:** YYYY-MM-DD
**Dependencies:** [List any upstream tables, notebooks, or datasets this depends on]
```

### Cell 2 — Parameters (Code)
Always use `notebookutils.notebook.entry_point.getParameters()` pattern so the notebook can be called from a pipeline with parameter injection.

```python
# ── Parameters ───────────────────────────────────────────────────────────────
# Default values used when running interactively.
# When called from a pipeline, these are overridden by the pipeline's parameters.

# notebookutils.notebook.exit() is used at the end to return output to caller.

ENVIRONMENT = "DEV"          # "DEV" or "PRD"
RUN_MODE    = "full"         # "full" or "incremental"
LOG_LEVEL   = "INFO"         # "DEBUG" or "INFO"

# Workspace + dataset IDs (set per environment) — fill in your own values
WORKSPACE_IDS = {
    "DEV": "<your-dev-workspace-guid>",
    "PRD": "<your-prd-workspace-guid>",
}
DATASET_IDS = {
    "DEV": "<your-dev-dataset-guid>",
    "PRD": "<your-prd-dataset-guid>",
}

workspace_id = WORKSPACE_IDS[ENVIRONMENT]
dataset_id   = DATASET_IDS[ENVIRONMENT]
```

### Cell 3 — Imports and Logging Setup (Code)
```python
# ── Imports ───────────────────────────────────────────────────────────────────
import logging
import sys
from datetime import datetime, timezone
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import *

# ── Logging ───────────────────────────────────────────────────────────────────
logging.basicConfig(
    level=getattr(logging, LOG_LEVEL, logging.INFO),
    format="%(asctime)s [%(levelname)s] %(message)s",
    datefmt="%Y-%m-%dT%H:%M:%S",
    stream=sys.stdout,
)
log = logging.getLogger(__name__)

log.info("Notebook started | env=%s | mode=%s", ENVIRONMENT, RUN_MODE)
run_start = datetime.now(timezone.utc)
```

### Cell 4 — Main Logic (Code)
All business logic goes here. Structure it with clear section comments:
```python
# ── [Section name] ────────────────────────────────────────────────────────────
```

Wrap the entire main logic block in try/except:
```python
try:
    # ── [Section 1] ───────────────────────────────────────────────────────────
    log.info("Starting [section]...")

    # ... your logic ...

    log.info("[Section] complete.")

    # ── [Section 2] ───────────────────────────────────────────────────────────
    # ...

except Exception as e:
    log.error("Notebook failed: %s", str(e), exc_info=True)
    notebookutils.notebook.exit(f"FAILED: {str(e)}")
    raise
```

### Cell 5 — Exit and Summary (Code)
```python
# ── Summary ───────────────────────────────────────────────────────────────────
run_end      = datetime.now(timezone.utc)
duration_sec = (run_end - run_start).total_seconds()

log.info("Notebook complete | duration=%.1fs | env=%s", duration_sec, ENVIRONMENT)

# Return a result summary to the calling pipeline (if invoked by pipeline).
# notebookutils.notebook.exit() value is surfaced in the pipeline's activity output.
notebookutils.notebook.exit(f"SUCCESS | duration={duration_sec:.1f}s | env={ENVIRONMENT}")
```

---

## Fabric-Specific Patterns

### Reading from Lakehouse (Delta table)
```python
df = spark.read.format("delta").load("abfss://[container]@[storage].dfs.core.windows.net/[path]")
# Or using shorthand in Fabric:
df = spark.read.format("delta").table("lakehouse.schema.table_name")
```

### Writing to Lakehouse (Delta table)
```python
(df.write
   .format("delta")
   .mode("overwrite")           # or "append" / "merge"
   .option("overwriteSchema", "true")
   .save("abfss://[container]@[storage].dfs.core.windows.net/[path]")
)
```

### Calling Another Notebook (from pipeline or parent notebook)
```python
result = notebookutils.notebook.run(
    "[notebook-name]",
    timeout_seconds=600,
    arguments={"ENVIRONMENT": ENVIRONMENT, "RUN_MODE": RUN_MODE}
)
log.info("Child notebook result: %s", result)
```

### Getting a Secret from Key Vault (or equivalent)
```python
client_secret = notebookutils.credentials.getSecret(
    "https://<your-keyvault-name>.vault.azure.net/",
    "<your-secret-name>"
)
```

### Semantic Model Refresh via REST API (Web Activity equivalent in notebook)
```python
import requests

def get_sp_token(tenant_id: str, client_id: str, client_secret: str) -> str:
    url = f"https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token"
    body = {
        "grant_type": "client_credentials",
        "client_id": client_id,
        "client_secret": client_secret,
        "scope": "https://analysis.windows.net/powerbi/api/.default",
    }
    resp = requests.post(url, data=body, timeout=30)
    resp.raise_for_status()
    return resp.json()["access_token"]

def trigger_dataset_refresh(workspace_id: str, dataset_id: str, token: str) -> None:
    url = f"https://api.powerbi.com/v1.0/myorg/groups/{workspace_id}/datasets/{dataset_id}/refreshes"
    headers = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    body = {"notifyOption": "NoNotification", "commitMode": "Transactional", "retryCount": 3}
    resp = requests.post(url, json=body, headers=headers, timeout=30)
    resp.raise_for_status()
    log.info("Refresh triggered | status=%d", resp.status_code)
```

---

## Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Notebook file | `[verb]_[subject]_[env].ipynb` | `refresh_sales_fact_dev.ipynb` |
| Variables | `snake_case` | `workspace_id`, `run_start` |
| DataFrames | `df_[subject]` | `df_sales`, `df_customers` |
| Temp views | `[subject]_tmp` | `sales_tmp` |
| Delta tables | `[schema].[subject]` | `silver.sales_fact` |
| Constants | `UPPER_SNAKE_CASE` | `ENVIRONMENT`, `RUN_MODE` |

---

## Anti-Patterns to Avoid

- Never hardcode client secrets, passwords, or connection strings — always use Key Vault or notebookutils.credentials
- Never use `df.collect()` on large datasets — use `.show()`, `.display()`, or aggregate first
- Never write `spark.sql("DROP TABLE ...")` without a dry-run flag
- Never use `print()` for logging — use the `log` logger so output is structured
- Never leave `display(df)` calls active in production notebooks — wrap with `if LOG_LEVEL == "DEBUG":`
- Never hardcode workspace or dataset IDs as bare strings — always use the environment dict pattern above

---

## Notebook Types — What to Generate

| Type | Key cells to include | Notes |
|---|---|---|
| Ingestion | Header, params, imports, source read + schema validation, write to Bronze/Silver, exit | Add row count log before and after |
| Transformation | Header, params, imports, read Silver, transform, write Gold, exit | Log record counts at each stage |
| Semantic model refresh | Header, params, imports, get SP token, trigger refresh, poll status, exit | Include retry loop for refresh status polling |
| Utility / helper | Header, params, imports, logic, exit | Keep under 100 lines; if longer, split |
| Data quality | Header, params, imports, read table, run assertions, log failures, exit with pass/fail | Use `assert` or custom validator; never silently pass |
