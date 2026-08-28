# Wind Turbine Data Pipeline POC

This project implements a data pipeline for processing wind turbine measurements.

The source data is daily CSV files with measurements for groups of turbines. 

The pipeline:

- ingests the CSV files
- loads the source data in Snowflake
- cleans and validates
- calculates daily statistics for each turbine
- identifies anomalies
- populates daily analytical tables

The assessment requires the use of Python and PySpark and any frameworks. For this POC PySpark is used for ingestion and Snowflake is used as a main platform. The transformation and business logic is implemented in SQL inside Snowflake.

In a production implementation I would use dbt for the transformation layer because of its support for modulararity, testing, documentation, lineage and source freshness checks. To keep this POC  time-boxed the SQL transformations are implemented directly in Snowflake Notebooks.

## Assumptions

For this POC:

- turbine_id uniquely identifies a turbine
- turbine_id + measurement_timestamp is source logical measurment key and it the pipeline uses it to to treat repeated processing of the same source data
- a turbine remains associated with the same source group
- power output is measured in MW
- wind direction allowed values are between 0 and 360
- negative wind speed is considered invalid
- negative power output is considered invalid
- missing/uncompleted readings are registered but not automatically populated
- daily statistics are calculated by calendar date based on timestamp
- duplicate source data wouldn’t create duplicates in cleaned measurements