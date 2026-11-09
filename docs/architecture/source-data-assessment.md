# Source Data Assessment — Wide World Importers

**Status:** Observed results from the laboratory run on 2026-11-09. This document records measured findings, not assertions about every future release of the dataset.

## Bronze sources

- Historical: `Files/bronze/wwi-raw-data/full/fact_sale`
- Incremental: `Files/bronze/wwi-raw-data/incremental/fact_sale_1y_incremental`
- Other full dimensions: `dimension_city`, `dimension_customer`, `dimension_date`, `dimension_employee`, `dimension_stock_item`
- Alternative historical set: `fact_sale_1y_full` (not merged; equality with `fact_sale` not exhaustively proven)

## Observed source metrics

| Metric | Historical | Incremental |
|---|---:|---:|
| Rows | 50,150,843 | 13,611,644 |
| Distinct `SaleKey` | 50,150,843 | 13,611,644 |
| Null `SaleKey` | 0 | 0 |
| Invoice period | Jan–Nov 2000 | Oct–Dec 2000 |

Incremental keys also present in historical: **8,962,661**. New keys: **4,648,983**.

## Reconciliation by invoice month

| Period | Source rows | Updates | Inserts | Unchanged |
|---|---:|---:|---:|---:|
| Oct 2000 | 4,448,043 | 0 | 0 | 4,448,043 |
| Nov 2000 | 4,514,618 | 4,514,618 | 0 | 0 |
| Dec 2000 | 4,648,983 | 0 | 4,648,983 | 0 |

Pre-MERGE comparison included all **20 non-key business fields**. In November, all 4,514,618 matched rows changed in `Quantity`, `TotalExcludingTax`, `TaxAmount`, `TotalIncludingTax`, and `TotalDryItems`. Other compared business columns showed no changes.

## Schema drift

Three decimal definitions differ between sources. Normalize `TotalExcludingTax`, `TaxAmount`, and `TotalIncludingTax` to `decimal(38,6)` before integration. Values passed observed null checks after normalization, but this is not a formal numeric overflow audit for every possible future batch.

## Important caveats

- The incremental feed is treated as authoritative for changed keys **for this laboratory**; no source change timestamp or CDC ordering is present in the contract.
- Source files are not partitioned by ingest batch in this exercise; production pipelines need batch tracking, source freshness logic, and concurrency control.
- Foreign-key coverage, distribution, monetary reconciliation, source license, and archival retention have not yet been independently audited.
- The comparison is within the supplied educational WWI data. Do not claim client or production business impact.
