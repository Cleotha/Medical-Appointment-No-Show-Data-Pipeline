# Medical Appointment No-Show Data Pipeline

## Overview

This project implements an end-to-end data engineering pipeline using **PySpark and Databricks** to ingest, clean, transform, and aggregate medical appointment data. The pipeline prepares appointment records for analysis and generates summary tables to investigate patterns associated with missed medical appointments.

The project follows a **Bronze, Silver, and Gold architecture** and uses Apache Parquet for data storage.

## Objectives

- Build a structured data pipeline using PySpark.
- Apply data quality checks and validation rules.
- Standardize column names and data types.
- Identify invalid ages and inconsistent appointment dates.
- Calculate appointment waiting times and categorize patients by age group.
- Generate Gold-layer summaries of appointment no-show rates.
- Store processed data as Parquet files in Databricks Volumes.
- Implement logging to track pipeline execution.

## Technology Stack

- **Language:** Python
- **Processing:** Apache Spark (PySpark)
- **Platform:** Databricks
- **Storage format:** Apache Parquet
- **Logging:** Python `logging` module
- **Data architecture:** Medallion architecture (Bronze, Silver, Gold)

## Dataset

The project uses the Medical Appointment No-Show dataset, `KaggleV2-May-2016.csv`, which contains appointment information such as:

- Patient and appointment IDs
- Gender and age
- Scheduled and appointment dates
- Neighbourhood
- Scholarship and health-condition indicators
- SMS reminder status
- Appointment attendance status

## Pipeline Architecture

### 1. Bronze Layer — Ingestion

The raw CSV dataset is ingested into a Spark DataFrame using an explicitly defined schema.

Key operations:
- Read the source CSV file from a Databricks Volume.
- Define column names and data types using `StructType` and `StructField`.
- Log ingestion progress and record counts.
- Save the ingested dataset in Parquet format.

### 2. Silver Layer — Cleaning and Transformation

The raw data is cleaned and transformed into a more analysis-ready dataset.

Key operations:
- Standardize column names.
- Cast columns to appropriate data types.
- Check for null patient and appointment IDs.
- Identify duplicate appointment IDs.
- Flag invalid ages outside the range of 0–100.
- Identify appointments scheduled after their appointment dates.
- Create a Boolean `Missed_Appointment` column.
- Calculate appointment waiting time in days.
- Create age groups and appointment weekday columns.
- Remove records flagged for invalid ages or inconsistent dates.
- Save the transformed dataset as partitioned Parquet files.

### 3. Gold Layer — Aggregation

The Gold layer produces summary tables using reusable PySpark aggregation logic.

Each summary includes the total number of appointments and the percentage of appointments missed.

| Gold summary | Grouping column |
|---|---|
| No-show rate by age group | `age_group` |
| No-show rate by neighbourhood | `Neighbourhood` |
| No-show rate by waiting time | `wait_days` |
| No-show rate by SMS reminder | `SMS_received` |
| No-show rate by scholarship status | `Scholarship` |

The no-show rate is calculated as:

`No-show rate (%) = (Missed appointments / Total appointments) × 100`

Rates are rounded to two decimal places for readability.

### 4. Data Storage

The pipeline stores its outputs in a Databricks Volume.

```text
medical_appointment/
├── ingest/
│   └── Raw ingested data in Parquet format
├── transform/
│   └── Cleaned and transformed data in partitioned Parquet
└── gold/
    ├── gold_no_show_by_age/
    ├── gold_no_show_by_neighbourhood/
    ├── gold_no_show_by_wait_time/
    ├── gold_no_show_by_sms/
    └── gold_no_show_by_scholarship/
```

The Silver dataset is partitioned by `Appointment_Day`. Gold summaries are saved as separate Parquet datasets.

## Logging and Monitoring

Python logging is used to track important pipeline events, including:

- Pipeline execution start
- Successful data ingestion
- Column standardization
- Data quality flag creation
- Transformation completion
- Gold summary generation

These log messages improve traceability and help with debugging.

## Data Quality Considerations

The pipeline investigates and handles several data quality issues:

- Invalid ages
- Inconsistent scheduling and appointment dates
- Duplicate appointment IDs
- Missing patient or appointment identifiers
- Inconsistent data types

Records flagged for invalid ages or inconsistent dates are excluded from the transformed dataset. Duplicate and null identifier checks are performed for investigation; they are not automatically removed by the current implementation.

## Running the Project

1. Open the notebook in Databricks.
2. Upload `KaggleV2-May-2016.csv` to the configured Databricks Volume.
3. Update the source and output paths if necessary.
4. Run the ingestion stage.
5. Execute the cleaning and transformation stage.
6. Generate the Gold summary tables.
7. Inspect the resulting Parquet datasets in the Volume.

## Potential Improvements

- Write Silver and Gold tables to PostgreSQL using JDBC.
- Add automated data quality tests with pytest.
- Introduce incremental processing instead of overwriting all outputs.
- Add exception handling and more detailed logging.
- Orchestrate pipeline execution using Apache Airflow.
- Deploy the pipeline using cloud infrastructure.
- Add dashboard visualizations for appointment no-show trends.

## Key Learning Outcomes

This project demonstrates practical experience with PySpark DataFrames, explicit schemas, transformations, aggregations, data quality validation, logging, Parquet storage, partitioning, and medallion architecture.

It also provides a foundation for building scalable data pipelines that prepare data for reporting and downstream analytics.
