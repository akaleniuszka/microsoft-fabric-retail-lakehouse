# Fact Sale — Silver Data Contract

**Status:** Sample-data rules verified; operational contract incomplete.

**Target:** `silver.fact_sale` (managed Delta table in the default schema-enabled Fabric Lakehouse).

**Grain:** One record per `SaleKey` in this laboratory source; business meaning beyond the sample definition remains to be documented.

## Rules

- `SaleKey` is a non-null, unique logical identifier (verified on both source snapshots and final Silver output).
- `InvoiceDateKey`, `Quantity`, and the three primary monetary fields must be non-null for the current sample.
- `Quantity` must be positive in the current sample (not necessarily a universal rule for returns or credits).
- `TotalExcludingTax`, `TaxAmount`, `TotalIncludingTax`: `decimal(38,6)` in Silver.
- Preserve remaining business columns and their existing types.
- `_ingestion_timestamp` and `_record_source` track which operation last wrote a Silver row; they are not a complete audit/change-data-capture history.

## Merge semantics

- Match on `SaleKey`.
- When matched: update all non-key business columns only if at least one differs using null-safe comparison.
- When unmatched: insert the record.
- When not present in source: do not delete target row.
- Preserve prior state for records not changed by the incoming batch.
- Treat incoming records as the authoritative version **as a laboratory assumption only**.
- Reject duplicate/null source keys; reject incompatible schemas.

## Verified first-run result

- Initial target: 50,150,843 rows (Delta version 0).
- MERGE: 4,514,618 updated; 4,648,983 inserted; 0 deleted (Delta version 1).
- Final target: **54,799,826** rows, matching distinct `SaleKey`; zero null keys.
- Read-only comparison of entire incremental feed to Silver: **0 missing keys; 0 changed business rows**.
- **Caution:** that is an idempotent-state check, not yet a second-execution test proving zero additional Delta changes.

## Pending

Foreign-key validation against dimension tables; batch manifest/watermark; audited source licenses; financial amount equations; integration and recovery tests; rerun audit of Delta `operationMetrics`.
