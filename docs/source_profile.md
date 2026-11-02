
# Day 1 – Source Data Profiling Report

**Project:** Flight Delay and Airport Operations  
**Phase:** Day 1 – Source Profiling  
**Purpose:** Understand source data quality, structure, duplicates, missing values and key relationships before Silver-layer transformations.

## 1. Source inventory

| Source | Format | Files / Tables | Records |
|---|---|---:|---:|
| Primary flight events | CSV | 12 files | 11,540 |
| Status updates | CSV | 1 file | 520 |
| Secondary events | CSV | 1 file | 310 |
| Airline reference | Azure SQL | 1 table | 110 |
| Airport reference | Azure SQL | 1 table | 150 |
| **Total** | | | **12,630** |

## 2. Primary flight event profiling

### 2.1 Record counts and duplicates

| Check | Expected | Actual | Result |
|---|---:|---:|---|
| Total primary records | 11,540 | 11,540 | PASS |
| Distinct flight IDs | 11,500 | 11,500 | PASS |
| Extra duplicate rows | 40 | 40 | PASS |
| Exact duplicate rows | 40 | 40 | PASS |

All 40 extra rows were identified as exact duplicates across the original 12 source columns.

Duplicates are retained in Bronze and will be handled in Silver.

### 2.2 Missing values

| Column | Expected missing values | Finding |
|---|---:|---|
| flight_type | 12 | 12 missing values |
| Other original source columns | 0 | No missing values |

The missing `flight_type` values are treated as a data-quality finding, not silently replaced during Bronze ingestion.

### 2.3 Schema evolution

| Batch | Source columns | Schema version |
|---|---:|---|
| 01–09 | 12 | Original schema |
| 10–12 | 13 | Evolved schema |

The additional column is `source_schema_version`.

Validation confirmed:

- The column exists in the Bronze table.
- Batches 10–12 contain `v2`.
- Earlier batches have NULL for this column.
- 2,540 rows contain the `v2` value.

### 2.4 Flight status distribution

| Flight status | Records |
|---|---:|
| FAILED | 2,979 |
| OPEN | 2,881 |
| COMPLETED | 2,856 |
| PENDING | 2,824 |
| **Total** | **11,540** |

No `CANCELLED` status was found in the primary flight event source.

### 2.5 Flight type distribution

| Flight type | Records |
|---|---:|
| INTERNATIONAL | 5,920 |
| DOMESTIC | 5,608 |
| NULL | 12 |
| **Total** | **11,540** |

### 2.6 Fare and delay profiling

| Metric | Expected profiling result |
|---|---:|
| Minimum fare | 100.16 |
| Median fare | Approximately 601.4 |
| 99th percentile fare | Approximately 1,092.9 |
| Maximum fare | 77,083.50 |
| Minimum delay | 0 |
| Average delay | Approximately 44.99 |
| Maximum delay | 90 |
| Extreme fare records above 2,350 | 35 |

The 35 extreme fare records are retained in Bronze for investigation and handling in Silver.

### 2.7 Timezone observation

The `timezone` column contains `USD` in all 11,540 primary records.

This appears to represent a currency code rather than a timezone identifier. It must not be used as a timezone for timestamp conversion.

## 3. Status updates profiling

| Check | Expected | Actual | Result |
|---|---:|---:|---|
| Total CDC records | 520 | 520 | PASS |
| Distinct flight IDs | 520 | 520 | PASS |
| COMPLETED | 176 | 176 | PASS |
| REVERSED | 173 | 173 | PASS |
| CANCELLED | 171 | 171 | PASS |
| No-op status changes | 45 | 45 | PASS |

The status updates are stored in `bronze_status_cdc`.

## 4. Secondary events profiling

| Check | Expected | Actual | Result |
|---|---:|---:|---|
| Secondary event records | 310 | 310 | PASS |
| Events without matching flight IDs | 0 | 0 | PASS |

The secondary event records are stored in `bronze_secondary_events`.

## 5. Reference data and key coverage

| Check | Expected | Actual | Result |
|---|---:|---:|---|
| Airline reference records | 110 | 110 | PASS |
| Airport reference records | 150 | 150 | PASS |
| Events with unknown airline IDs | 0 | 0 | PASS |
| Events with unknown airport IDs | 0 | 0 | PASS |
| CDC records without matching flights | 0 | 0 | PASS |

Airline and airport reference tables were loaded from Azure SQL using JDBC and saved as Delta tables.

## 6. Bronze ingestion reconciliation

| Bronze table | Expected rows | Actual rows | Result |
|---|---:|---:|---|
| bronze_flight_events | 11,540 | 11,540 | PASS |
| bronze_status_cdc | 520 | 520 | PASS |
| bronze_secondary_events | 310 | 310 | PASS |
| airline | 110 | 110 | PASS |
| airport | 150 | 150 | PASS |
| **Total** | **12,630** | **12,630** | **PASS** |

## 7. Overall findings

1. The primary source contains 40 extra exact duplicate rows.
2. There are 12 missing optional `flight_type` values.
3. There are 35 extreme fare records.
4. The primary source schema evolves in batches 10–12.
5. The `timezone` column contains `USD`, so it cannot be treated as a timezone.
6. The primary flight event source does not contain a `CANCELLED` status.
7. The CDC source contains `COMPLETED`, `REVERSED` and `CANCELLED` status updates.
8. All checked airline, airport and flight key relationships have zero unmatched records.

## 8. Conclusion

Source profiling and Bronze ingestion were completed for the supplied datasets.

The actual record counts matched the expected QA counts. Identified data-quality issues and source anomalies have been documented for downstream Silver-layer processing.
