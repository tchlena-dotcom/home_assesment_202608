# Wind Turbine Data Pipeline POC

This project implements a data pipeline for processing wind turbine measurements.

The source data is daily CSV files with measurements for groups of turbines. 

The pipeline:

- ingests the CSV files into staging schema in Snowflake
- cleans, validates source data, calculates anomaly and load into cleaned schema
- calculates daily statistics for each turbine and save into mart schema

The assessment requires the use of Python and PySpark and any frameworks. For this POC PySpark is used for ingestion and transformation and Snowflake is used as a main platform. 

In a production implementation I would use dbt for the transformation layer because of its support for modulararity, testing, documentation, lineage and source freshness checks. To keep this POC time-boxed the pipeline is implemented in Snowflake Notebooks. 

## Project strucure
- readme.md
- design.md
- notebooks
  - ingestion.ipynb
  - transformation.ipynb
- sql files
  - poc_setup.sql
  - tables.sql

## Assumptions

For this POC:

- turbine_id uniquely identifies a turbine
- turbine_id + measurement_timestamp is source logical measurment key and it the pipeline uses it to to treat repeated processing of the same source data
- a turbine remains associated with the same source group
- wind direction allowed values are between 0 and 360
- negative wind speed is considered invalid
- negative power output is considered invalid
- missing/uncompleted/invaled readings are registered in staging but not automatically populated
- daily statistics are calculated by calendar date based on measurement timestamp
- duplicate source data wouldn’t create duplicates in cleaned measurements
- anomality detection is calculated and saved in cleaned table but not filtered out and daily statistics are calculated based on all metrics
