---
name: code-review
description: Perform an expert code review for a Microsoft Fabric data engineering stack. Use when the user pastes code and wants a structured review. Covers Python/PySpark, SQL, DAX, Power Query/M, and Fabric pipeline expressions. Returns findings by severity with specific line references, explanations, and concrete fixes — not generic advice.
---

# Skill: /code-review

You are performing a senior-level code review for a Microsoft Fabric data engineering stack. Every review must be structured, specific, and actionable. No generic advice — every finding must reference a specific line, pattern, or location and include a concrete fix.

---

## Review Output Format

Always produce output in this exact structure:

### 1. Summary
One paragraph: what the code does, what language/context it is, overall quality signal (Clean / Minor issues / Needs work / Block — do not ship).

### 2. Findings Table

| # | Severity | Location | Issue | Fix |
|---|---|---|---|---|
| 1 | 🔴 Critical | line X | Description | Specific fix |
| 2 | 🟠 Major | line X | Description | Specific fix |
| 3 | 🟡 Minor | line X | Description | Specific fix |
| 4 | 🔵 Info | line X | Description | Suggestion |

Severity definitions:
- **Critical** — will cause failure, data loss, security breach, or incorrect results in production. Do not ship.
- **Major** — degrades performance significantly, causes intermittent failures, or breaks under edge cases. Fix before shipping.
- **Minor** — poor practice, maintenance risk, or readability issue. Fix when convenient.
- **Info** — improvement suggestion, style note, or observation. Optional.

### 3. Revised Code
Provide a corrected version of the code with all Critical and Major issues fixed. Mark changes with inline comments: `# FIXED: [reason]`. Do not rewrite sections that didn't need changing.

### 4. Verdict
One line: `PASS`, `PASS WITH NOTES`, or `BLOCK`. If BLOCK, state exactly what must be fixed before shipping.

---

## Language-Specific Review Criteria

### Python / PySpark (Fabric notebooks and scripts)

**Performance — always check:**
- `df.collect()` on large datasets — forces entire dataset into driver memory. Flag as Critical if in a loop; Major otherwise. Fix: use `.first()`, `.take(n)`, aggregations, or Delta reads instead.
- `df.count()` called multiple times — triggers a full scan each time. Fix: cache the df or store count once.
- Shuffle-heavy operations without partition awareness (`groupBy`, `join`, `orderBy`) — check if data is partitioned on the join/group key.
- Missing `.cache()` or `.persist()` on DataFrames used more than once in the same job.
- Schema inference on large files (`inferSchema=True`) — triggers an extra full scan. Fix: define schema explicitly.

**Correctness — always check:**
- Null handling: operations on columns that may contain nulls without explicit null guards (`F.coalesce`, `F.when(...).otherwise(...)`, `.fillna()`).
- Date/timezone handling: naive datetime objects in a distributed context. Always use `timezone.utc` or Spark's `F.to_utc_timestamp`.
- Type coercions: implicit string-to-int casts that will fail silently on bad data.
- Schema drift: reading Delta tables without schema validation — new columns added upstream will silently change downstream output.
- Off-by-one in window functions or date ranges.

**Security — always check:**
- Hardcoded secrets, passwords, connection strings, or client IDs — Critical. Fix: use `notebookutils.credentials.getSecret()` or environment variable.
- `eval()` or `exec()` on any user-supplied or external input — Critical.
- SQL string concatenation with external input (injection risk) — Critical. Fix: use parameterised queries.
- Logging sensitive values (passwords, tokens, PII fields) — Major.

**Maintainability:**
- Magic numbers or hardcoded IDs (workspace IDs, dataset IDs) without named constants.
- Functions longer than ~50 lines without a clear decomposition reason.
- Missing error handling — bare `except:` or `except Exception as e: pass`.
- `print()` used instead of structured logging.
- No `notebookutils.notebook.exit()` at the end of Fabric notebooks.

### SQL (T-SQL, Spark SQL, Fabric warehouse)

**Performance:**
- `SELECT *` in production queries — always enumerate needed columns.
- Missing `WHERE` clause on large tables — full scans.
- Non-SARGable predicates: `WHERE YEAR(date_col) = 2024` prevents index use. Fix: `WHERE date_col >= '2024-01-01' AND date_col < '2025-01-01'`.
- Implicit type conversions in `JOIN` or `WHERE` conditions — prevents predicate pushdown.
- Correlated subqueries inside `SELECT` or `WHERE` that execute per row — replace with `JOIN` or window function.
- `ORDER BY` without `TOP`/`LIMIT` on large result sets.

**Correctness:**
- `NULL` comparison using `= NULL` instead of `IS NULL` — always silently returns no rows.
- Outer join columns used in `WHERE` clause — silently converts `LEFT JOIN` to `INNER JOIN`.
- `COUNT(*)` vs `COUNT(column)` — different behaviour on NULLs; flag if it looks like the author didn't intend it.
- `UNION` vs `UNION ALL` — `UNION` deduplicates (slower); flag if deduplication is not intentional.
- Implicit date conversions that are locale-dependent.

**Security:**
- Dynamic SQL with string concatenation — injection risk. Fix: `sp_executesql` with parameters.
- `GRANT` statements in migration scripts that are broader than necessary.

### DAX (Power BI semantic models)

**Performance — the most common DAX mistakes:**
- `SUMX` / `COUNTX` / iterator functions over large tables when `SUM` / `COUNT` would work — iterators evaluate row-by-row. Flag if table has >1M rows.
- Calculated columns that could be measures — calculated columns are computed at refresh and stored; measures are computed at query time. Storing aggregations as calculated columns wastes memory.
- Bidirectional cross-filter (`Both`) on relationships without a documented reason — causes ambiguous filter paths and unpredictable results.
- `CALCULATE` with multiple filter arguments — each filter is `AND`-ed, not `OR`-ed. Flag if the author appears to expect `OR` behaviour.
- `ALL()` vs `ALLEXCEPT()` vs `ALLSELECTED()` — wrong choice is the most common cause of incorrect totals.
- `RELATED()` used in a measure (only valid in calculated columns and row context) — will error.
- Context transition not accounted for when `CALCULATE` is inside an iterator.
- `IF(ISBLANK(...))` wrapping measures — usually a sign the base measure returns BLANK when it should return 0.

**Correctness:**
- Filter direction: check that the relationship filter direction matches the intended aggregation direction.
- Time intelligence functions (`SAMEPERIODLASTYEAR`, `DATESYTD`) require a continuous date table marked as a date table — flag if date table status is unknown.
- `DIVIDE(numerator, denominator)` vs `/` — bare division errors on zero; always use `DIVIDE`.
- Measure name matches a column name — DAX will silently resolve to the column in some contexts.

### Power Query / M (dataflows, semantic model transforms)

**Query folding — always check:**
- Any step that breaks folding (adding an index column, using `Table.Buffer`, certain custom column expressions) means all subsequent steps run in-memory. Flag if folding breaks early and there are heavy operations downstream.
- `Table.Buffer()` used without a documented reason — forces all data into memory.
- Steps that could be pushed to the source (filters, column removal, type casting) placed after a step that breaks folding.

**Performance:**
- `Table.NestedJoin` followed by `Table.ExpandTableColumn` on large tables — verify folding; if not folding, this is a full in-memory merge.
- Unused query steps that are still evaluated.
- `List.Generate` or recursive functions in M — almost always a performance problem at scale.

**Correctness:**
- Hardcoded locale in `Number.From`, `Date.From`, etc. — use explicit locale parameter.
- Missing error handling on data source connections — one bad row can fail the entire refresh.

### Fabric Pipeline Expressions (Data Factory / ADF expression language)

- String concatenation to build file paths without null guards — `concat(variables('path'), '/')` fails if `path` is null.
- `@activity('name').output.value` without a null check — fails if upstream activity produced no output.
- Hardcoded environment names or workspace IDs in pipeline parameters instead of pipeline-level variables.
- Missing `dependsOn` with correct `Succeeded`/`Failed` conditions — activities that should only run on success may run regardless.
- `forEach` activities with no `batchCount` limit — can spawn too many parallel activities.

---

## Environment & Secrets Hygiene — Always Check

- **Environment handling:** Is the code parameterised for DEV vs PRD? Hardcoded workspace/dataset IDs should live in a per-environment config dict, never as bare strings inline.
- **Auth pattern:** Any OAuth 2.0 user-delegated token usage for automated processes — flag as Major. Should be a Service Principal instead.
- **Semantic model refresh:** Native "Semantic Model Refresh" pipeline activities using the pipeline's default managed identity carry a fixed expiry (commonly ~90 days) — flag as a note; a Web Activity calling the REST API directly with a Service Principal token is the more durable pattern.
- **Secret storage:** Any client secret, password, or key stored outside a secrets vault (Key Vault or equivalent) — Critical.

---

## What NOT to do in a review

- Do not rewrite code that works and has no issues just to apply a different style.
- Do not flag style preferences (variable naming, spacing) as Major or Critical.
- Do not suggest adding abstractions or refactoring for hypothetical future requirements.
- Do not add "it would be nice to also..." suggestions beyond the Info severity level.
- Keep the revised code focused on fixing findings — do not expand scope.
