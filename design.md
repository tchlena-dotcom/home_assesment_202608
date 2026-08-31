# Wind Turbine POC Design

## 1. Overview

The solution is implemented in **Snowflake using PySpark and SQL**. It processes CSV files containing wind turbine measurements through three logical layers:

**STAGING → CLEANED → MART**

The pipeline is designed to support incremental processing when the same source files are updated with new daily data.

## 2. Architecture

### STAGING

Raw CSV files are stored in the Snowflake internal stage:

`STAGING.WIND_TURBINE`

The ingestion notebook reads the files using PySpark and loads the raw values into:

`STAGING.STG_TURBINE_MEASUREMENT`

Raw source values are intentionally stored as `VARCHAR` so validation and type conversion happen during transformation rather than ingestion.

Additional metadata is captured:

- `SOURCE_FILE`
- `LOADED_DATETIME`

`STAGING.FILE_LOAD_HISTORY` tracks source files using filename, modification timestamp, checksum and processing status.

File statuses move through:

`PROCESSING → LOADED → COMPLETED`

This allows the pipeline to identify new or changed files and avoid unnecessarily reprocessing unchanged files.

## 3. Data Cleaning and Transformation

The transformation notebook reads only data belonging to files currently marked as `LOADED`.

PySpark performs:

- data type validation and conversion
- rejection of null or invalid values
- validation of wind speed and power output as non-negative
- validation of wind direction between 0 and 360 degrees
- derivation of `MEASUREMENT_DATE`

Cleaned measurements are merged into:

`CLEANED.TURBINE_MEASUREMENT`

using `(MEASUREMENT_TIMESTAMP, TURBINE_ID)` as the logical key, making processing idempotent when previously loaded measurements are received again.

A small `CLEANED.TURBINE` table maintains the known turbine IDs and their associated source files.

## 4. Anomaly Detection

For every turbine and measurement date the pipeline calculates:

- daily average power output
- daily standard deviation of power output

A measurement is flagged as an anomaly when its power output falls outside:

`Daily Average ± 2 × Daily Standard Deviation`

The result is stored as `IS_ANOMALY` in the cleaned measurement table.

## 5. Daily Statistics

Daily turbine-level statistics are calculated and merged into:

`MART.DAILY_TURBINE_STATS`

The mart contains:

- measurement count
- minimum power output
- maximum power output
- average power output

The grain of the table is:

**one row per turbine per day**.

**Assumptions**

- It is assumed that when a source file is updated for a given day, it contains the complete set of available measurements for that day rather than only a partial subset of hours.
- If partial-day updates are possible, the production implementation should recalculate the affected turbine/day statistics using both the newly received measurements and the previously loaded historical measurements from the cleaned layer.

## 6. POC Infrastructure

The POC uses:

- Snowflake database `WIND_TURBINE_POC`
- `STAGING`, `CLEANED`, and `MART` schemas
- an X-Small auto-suspending warehouse
- a dedicated `POC_ETL_ROLE`
- a Snowflake internal stage
- Snowflake Notebooks with PySpark/Snowpark connectivity

## 7. Assumptions

- Source files may retain the same filename while their contents change daily
- File checksum and modification timestamp are therefore used to identify changed files
- Invalid or missing measurements are removed rather than imputed in this POC
- Processing is incremental at file level and idempotent at cleaned measurement level
- The implementation prioritises clarity and testability appropriate for a proof of concept

This document describes the design of the wind turbine data pipeline POC

The design deliberately separates ingestion, cleaning and agretgation into independent layers

##