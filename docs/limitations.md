
# Day 1 – Project Limitations and Known Issues

**Project:** Flight Delay and Airport Operations

This document records limitations identified during source profiling and initial Bronze ingestion.

## 1. Source data limitations

### 1.1 Timezone information

The primary flight event dataset contains a column named `timezone`.

However, all 11,540 records contain the value `USD`.

This appears to be a currency code rather than a timezone identifier.

Impact:
- The column cannot reliably be used for timezone conversion.
- Timestamp normalisation based on this field is not possible.
- The original value is preserved in Bronze.

### 1.2 Missing flight type values

There are 12 records with missing `flight_type` values.

Impact:
- These records require a documented data-quality treatment in Silver.
- No replacement values are assumed during Bronze ingestion.

### 1.3 Duplicate flight records

The primary source contains 40 extra exact duplicate rows.

Impact:
- Bronze retains all source records.
- Deduplication will be implemented in Silver using the agreed business rules.

### 1.4 Extreme fare values

There are 35 records with fare values above 2,350.

Impact:
- These records are flagged for investigation.
- They are not automatically deleted or modified in Bronze.
- A Silver-layer quality rule will determine their treatment.

### 1.5 Status interpretation

The CDC source contains the status `REVERSED`.

The supplied source documentation does not establish its exact business meaning.

Impact:
- Its meaning must not be assumed.
- Business interpretation should be confirmed before applying CDC consolidation rules.

### 1.6 No schedule revision dataset

The supplied project files do not include a separate schedule revision dataset.

Impact:
- Schedule-revision-based temporal analysis cannot be implemented using the supplied sources alone.
- Any future requirement for this analysis would need an additional source.

## 2. Ingestion and implementation limitations

### 2.1 Static ingestion of CDC and secondary events

The primary flight event dataset uses Databricks Auto Loader.

The status updates and secondary events datasets were loaded using static CSV ingestion into Delta tables.

Impact:
- The current implementation validates the supplied snapshots.
- Continuous or incremental ingestion for these two sources has not yet been implemented.

### 2.2 Auto Loader recovery testing

Auto Loader uses configured schema and checkpoint locations.

A replay-safety test was performed by rerunning the ingestion without adding new files.

The Bronze row count remained unchanged at 11,540.

Impact:
- Reprocessing of already completed files was not observed during the replay test.
- A separate failure-and-recovery simulation has not yet been performed.

### 2.3 Bronze transformation policy

Bronze is intended to preserve source information with ingestion metadata.

Business transformations, deduplication and CDC consolidation are deferred to Silver.

Impact:
- Bronze data should not be treated as the final business-ready dataset.

## 3. Infrastructure and environment

### 3.1 Unity Catalog storage

The project uses an external volume:

`/Volumes/capstone_3flight/flight_schema/ext_capstone3_vol`

Source files and Auto Loader state directories are organised within this volume.

### 3.2 Reference data connectivity

Airline and airport reference entities were successfully read from Azure SQL using JDBC and saved as Delta tables in Unity Catalog.

The current reference tables contain:

- Airline: 110 records.
- Airport: 150 records.

## 4. Items to address in later phases

| Item | Planned phase |
|---|---|
| Missing flight type handling | Silver |
| Duplicate removal | Silver |
| Extreme fare handling | Silver |
| CDC consolidation | Silver |
| Status transition validation | Silver |
| Secondary event integration | Silver |
| Business-level analytics | Gold |
| Additional schedule data, if required | Future source integration |

## 5. Conclusion

The identified limitations are documented rather than silently corrected.

The Bronze layer preserves the supplied source information, while subsequent Silver and Gold processing will apply documented business and data-quality rules.
