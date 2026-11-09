# Microsoft Fabric Retail Lakehouse

An end-to-end **personal data engineering portfolio** project using Microsoft Fabric, PySpark, Delta Lake, and a Medallion-inspired architecture. Built with public educational sample data in an authorized Fabric lab; **not a production or client deployment**.

## Architecture

Public Wide World Importers Parquet → **Bronze** (raw files in Lakehouse) → **Silver** (normalized Delta facts with a conditional MERGE) → **Gold** (planned dimensional model).

Fabric Data Factory pipelines ingest raw data; PySpark normalizes decimal schema drift, validates critical columns, and reconciles existing versus incoming `SaleKey` values.

## Verified results (original Fabric working notebooks, 2026-11-09)

| Measure | Verified result |
|---|---:|
| Historical source rows | 50,150,843 |
| Incremental source rows | 13,611,644 |
| Existing rows updated by Delta MERGE | 4,514,618 |
| New rows inserted | 4,648,983 |
| Final `silver.fact_sale` rows | **54,799,826** |
| Final distinct `SaleKey` values | **54,799,826** |
| Missing or differing incremental business rows after MERGE | **0** |

A separate four-row synthetic test executed in Fabric demonstrated 1 UPDATE, 1 INSERT, unchanged rows, and an unchanged final state after a repeated MERGE. The rerun created no additional Delta version in that observed execution.

**Verification boundary:** The original exploratory notebooks produced the measurements above. The refactored published notebooks and the sanitized automated-style test notebook **have not yet been independently executed end to end**.

## Repository structure

- `docs/architecture/solution-design.md` — high-level solution design.
- `docs/architecture/source-data-assessment.md` — source profiling and reconciliation.
- `docs/architecture/fact-sale-contract.md` — Silver data contract.
- `docs/PROJECT_STATUS.md` — verified progress, remaining work, limitations.
- `notebooks/silver/01_initialize_fact_sale.ipynb` — one-time historical Delta initialization.
- `notebooks/silver/02_merge_incremental_fact_sale.ipynb` — conditional incremental MERGE.
- `notebooks/tests/test_delta_merge_idempotency.ipynb` — isolated synthetic MERGE test.

## Prerequisites and execution order

1. Obtain authorized access to a Microsoft Fabric capacity and a schema-enabled Lakehouse.
2. Create a `silver` schema and stage the WWI Parquet source files in `Files/bronze/wwi-raw-data/full/` and `Files/bronze/wwi-raw-data/incremental/`.
3. Attach the Lakehouse to the notebook session, then run `01_initialize_fact_sale.ipynb` **only when the target table does not exist**.
4. Run `02_merge_incremental_fact_sale.ipynb` after initial validation. Check its results and Delta history before rerunning if an error or connection interruption occurs.
5. For low-cost independent verification, create a `silver_test` schema and run `test_delta_merge_idempotency.ipynb`. This creates a uniquely named temporary Delta table and optionally drops it after assertions pass.

The working sample contains fixed row-count assertions. Adjust those only if the source dataset has been intentionally changed and its quality contract has been revalidated. Never run the initialization notebook against an existing populated Silver table.

## Engineering decisions and limitations

- `SaleKey` is a validated logical unique key for the tested sample; no physical primary-key constraint is claimed.
- The incremental batch is treated as authoritative **by laboratory convention** because there is no verified source CDC ordering timestamp.
- October overlaps without changes; November contains source updates; December contains new keys. No rows are deleted simply because they are absent from a batch.
- Three monetary columns are normalized to `decimal(38,6)` prior to merge.
- The dataset license/redistribution conditions should be checked before hosting any sample data in the public repository. **Raw data is not included here.**
- Spark admission can be constrained on a shared capacity (HTTP 430 observed during development).
- Remaining work: independent validation of refactored notebooks, batch manifests/watermarks, automated CI tests, referential-integrity checks, Silver dimensions, Gold model, and deployment automation.

## Security

No corporate data, client data, workspace IDs, tenant identifiers, connection credentials, or execution tokens should be committed. Public artifacts must be reviewed before publishing.

## References

- [Microsoft Fabric documentation](https://learn.microsoft.com/en-us/fabric/)
- [Wide World Importers sample overview](https://learn.microsoft.com/en-us/sql/samples/wide-world-importers-what-is)
