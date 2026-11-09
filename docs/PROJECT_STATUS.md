# Project Status

**As of:** 2026-11-09

## Completed and observed in Microsoft Fabric

- Authorized workspace and Lakehouse created (`lh_retail_dev`).
- Full and incremental Bronze Parquet copied from the public WWI sample source.
- Historical Copy Data: 77 files, 739,738,895 bytes, 38 seconds copy duration.
- Incremental Copy Data: 15 files (including `_SUCCESS`), 165,546,086 bytes, 21 seconds copy duration.
- Schema drift reconciled in PySpark; data-quality checks passed for tested critical columns.
- `silver.fact_sale` created (Delta v0, 50,150,843 rows).
- Incremental MERGE completed (Delta v1, 4,514,618 updates, 4,648,983 inserts).
- Final unique keys: 54,799,826; missing and changed source business rows after MERGE: 0.

## Engineering/operations observations

- A Spark session was previously rejected with `TooManyRequestsForCapacity` (HTTP 430); it later started successfully. Shared capacity may become constrained again.
- First-run notebook was an exploratory working file. Repo-ready notebooks here are **refactored and not yet independently executed**. Do not imply these refactored notebooks have been tested in Fabric.

## Next

1. Review and publish sanitized source notebooks and documentation.
2. Run refactored notebooks in an isolated test Lakehouse (never rerun one-time initialization against existing Silver table).
3. Confirm Delta MERGE rerun produces zero updates/inserts and investigate transaction-version behavior.
4. Implement automated quality tests and pipeline orchestration.
5. Build Silver dimensions, then Gold dimensional model.
