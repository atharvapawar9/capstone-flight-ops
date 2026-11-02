
# Day 1 – Technical Decision Log

**Project:** Flight Delay and Airport Operations  
**Phase:** Day 1 – Source Profiling and Bronze Ingestion

## D01 – Source grain

The source grain is defined as one event row per `flight_id` per source arrival.

A `flight_id` can therefore appear more than once in the source data. Bronze preserves these arrivals rather than removing duplicates during ingestion.

## D02 – Data architecture

The project follows a Medallion Architecture:

- Bronze: raw source data ingested into Delta tables.
- Silver: data cleaning, standardisation, deduplication and CDC consolidation.
- Gold: business-ready data for analytics and reporting.

Day 1 focuses on source profiling and Bronze ingestion.

## D03 – Unity Catalog organisation

The project uses:

- Catalog: `capstone_3flight`
- Schema: `flight_schema`
- External volume: `ext_capstone3_vol`

Source files are stored in separate folders for primary events, status updates and secondary events.

## D04 – Primary source ingestion

The 12 primary flight CSV batches are ingested using Databricks Auto Loader.

The Bronze target is:

`capstone_3flight.flight_schema.bronze_flight_events`

## D05 – Duplicate handling

The primary source contains 40 extra exact duplicate rows across 11,540 records.

Duplicates are retained in Bronze because Bronze preserves source arrivals. Deduplication is deferred to Silver.

## D06 – Schema evolution

Batches 10, 11 and 12 introduce the additional `source_schema_version` column.

Auto Loader uses `addNewColumns` schema evolution mode to capture this new column.

The Bronze table contains the column, with `v2` populated for the later batches.

## D07 – Ingestion metadata

The primary Bronze table records:

- `_source_file`: source file path.
- `_ingested_at`: ingestion timestamp.

These fields provide source traceability and ingestion auditing.

## D08 – Checkpointing and replay safety

Auto Loader uses a persistent checkpoint location to track processed files.

A replay test was performed without adding new files.

The Bronze row count remained 11,540 before and after replay, confirming that the rerun did not add duplicate records.

## D09 – CDC source ingestion

The status updates CSV is loaded into:

`capstone_3flight.flight_schema.bronze_status_cdc`

The source contains 520 records. CDC consolidation and business-rule application are deferred to Silver.

## D10 – Secondary event ingestion

The secondary events CSV is loaded into:

`capstone_3flight.flight_schema.bronze_secondary_events`

The source contains 310 records. Further transformation is deferred to Silver.

## D11 – Reference data ingestion

The airline and airport reference entities were read from Azure SQL using JDBC and saved as Delta tables in Unity Catalog.

The resulting tables contain 110 airline records and 150 airport records.

## D12 – Source anomalies are preserved for downstream processing

Profiling identified 12 missing optional `flight_type` values and 35 extreme fare records.

These records are not silently modified during Bronze ingestion. Their handling will be addressed through documented Silver-layer quality rules.
