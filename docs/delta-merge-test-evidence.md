# Portfolio test evidence — Delta MERGE

## Observed in the original Fabric notebook

- Initial synthetic table: keys 1, 2, 3.
- First MERGE: one changed key (2), one unchanged matched key (3), and one newly inserted key (4).
- Delta history after the first MERGE: version 1; one UPDATE, one INSERT, zero DELETE.
- Second identical MERGE: completed; Delta history remained at version 1 in the observed run.
- Final state: 4 rows and exact expected values; final assertion PASSED.

## Scope and qualification

This verifies the **observed original test**, not yet the newly refactored notebook. The repeated MERGE produced no new Delta history entry. Metrics displayed for version 1 belong to the first MERGE and must not be mistaken for metrics of the second run.

The sanitized test notebook generates a unique `silver_test` table per execution and tests row-level outcomes; it has not been independently executed in Fabric yet. This does not establish correctness under concurrent writes or production fault conditions.
