# Medical Appointment Data Cleaning Pipeline

## Project Overview

This project uses PySpark and Databricks to build a data pipeline for medical appointment records. It processes the Kaggle Medical Appointment No-Show dataset, applies data-quality checks, engineers useful features, and generates analytical summaries to explore patterns in missed appointments.

## Objectives

- Ingest medical appointment data from CSV.
- Apply a defined schema and standardise column names.
- Identify invalid ages and appointment dates.
- Engineer wait-time buckets, age groups, and appointment weekdays.
- Calculate appointment no-show rates across relevant categories.
- Store processed data in Parquet format using a Bronze/Silver/Gold-style architecture.

## Tech Stack

- Python
- Apache Spark (PySpark)
- Databricks
- Spark SQL functions
- Parquet
- Git and GitHub

## Pipeline Architecture

### 1. Ingestion Layer

- Reads the source CSV dataset from a Databricks Unity Catalog Volume.
- Applies an explicit Spark schema.
- Records pipeline progress using Python logging.
- Stores the ingested data in Parquet format.

### 2. Cleaning and Transformation Layer

- Standardizes column names, including `Patient_ID`, `Appointment_ID`, `Scheduled_Day`, and `SMS_Received`.
- Identifies invalid ages, including ages below zero or above 100.
- Identifies appointments scheduled after their appointment date.
- Removes records flagged by the configured age and date validation rules.
- Creates `Missed_Appointment` from the original `No-show` field.
- Calculates `wait_days` between scheduling and appointment dates.
- Groups waiting periods into four categories:
  - Same day
  - 1–7 days
  - 8–30 days
  - 30+ days
- Creates age groups and extracts appointment weekdays.
- Saves the transformed data as partitioned Parquet.

### 3. Gold Aggregation Layer

Creates summary datasets containing total appointments and no-show rates by:

- Age group
- Neighbourhood
- Wait-time bucket
- SMS reminder status
- Scholarship status

No-show rate is calculated as the percentage of appointments marked as missed within each group.

### 4. Storage Layer

The pipeline stores its outputs in separate directories within the medical appointment project path:

- `ingest/` — ingested data
- `transform/` — cleaned and transformed data
- `gold/` — analytical summary datasets

Parquet is used for columnar storage and compatibility with Spark-based analytical workflows.

## Data Quality and Feature Engineering

| Area | Rule or transformation |
|---|---|
| Age | Flag values below 0 or above 100 |
| Appointment dates | Flag appointments earlier than their scheduled dates |
| Wait time | Calculate the difference in calendar days |
| Wait-time buckets | Same day, 1–7, 8–30, and 30+ days |
| Age groups | Under 18, 18–35, 36–50, 51–65, and above 65, with an Unknown category for nulls |
| Appointment weekday | Extract the weekday from the appointment date |
| No-show indicator | Convert `Yes`/`No` values into a Boolean indicator |

## Findings

### Row-count summary

| Metric | Records |
|---|---:|
| Initial records | 110,527 |
| Records dropped | 13 |
| Final records | 110,514 |

Only 13 records were removed by the configured age and appointment-date validation rules.

### Key findings

1. **SMS reminders:** Appointments with an SMS reminder had a no-show rate of **27.58%**, compared with **16.70%** for appointments without a reminder. This is an observed association and does not establish that reminders increase no-shows.

2. **Waiting time:** Same-day appointments had the lowest no-show rate at **4.65%**. The rate was **24.14%** for 1–7 days, **31.72%** for 8–30 days, and **33.00%** for waits longer than 30 days.

3. **Age groups:** Patients aged **18–35** had the highest observed no-show rate at **23.81%**, followed by patients under 18 at **21.90%**. Patients above 65 had the lowest rate among the reported age groups at **15.51%**.

4. **Scholarship status:** Appointments for patients with scholarship status `1` had a no-show rate of **23.74%**, compared with **19.80%** for patients with scholarship status `0`.

### Interpretation

The results show that longer waiting periods were associated with higher observed no-show rates, while same-day appointments had a substantially lower rate. No-show rates also varied across age groups and scholarship status.

These are descriptive findings from the processed dataset, not proof of causation. Other factors may influence the observed patterns.


## How to Run

1. Open the notebook in Databricks.
2. Ensure the CSV dataset is available at the configured Unity Catalog Volume path.
3. Run the notebook cells in order, from ingestion through the Gold output stage.
4. Review the logged row counts and data-quality results.
5. Inspect the Gold summary DataFrames and the saved Parquet outputs.

## Future Improvements

- Add automated data-quality tests.
- Implement more robust handling of null values and duplicate appointment IDs.
- Add pipeline orchestration using Apache Airflow.
- Integrate cloud storage and processing services using AWS.
- Add automated monitoring and alerting.
- Publish the Gold summaries to a reporting or visualization tool.

## Learning Outcomes

This project demonstrates practical experience with PySpark DataFrame transformations, explicit schemas, data-quality validation, feature engineering, aggregations, logging, Parquet storage, and building a multi-stage data pipeline in Databricks.
