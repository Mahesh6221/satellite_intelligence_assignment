# 🌍 Satellite Intelligence – Production Data Engineering Pipeline (Databricks + PySpark)

## 🚀 Project Overview

This project is a **Production-style data engineering pipeline** built using **PySpark on Databricks**, designed to simulate real-world agricultural satellite intelligence systems used in precision farming, crop monitoring, and agronomic analytics.

It processes messy agricultural field data (sensor readings + parcel metadata) and converts it into clean, structured datasets for analytics.

It processes raw agricultural sensor data and parcel metadata, performs robust data cleaning, and generates analytics-ready datasets using a **Medallion Architecture (Bronze → Silver → Gold)**.

In real-world systems, agricultural data is:
- Missing and incomplete
- Noisy and inconsistent
- Sensor-failure prone
- Time-series dependent

This pipeline demonstrates how to handle such data using scalable Spark-based architecture.


---

# 📁 Project Structure

```text
satellite-intelligence-pipeline/
│
├── Project/
│   └── satellite_intelligence_pipeline.ipynb
│
├── volumes/
│   └── satellite_intelligence/
│       └── default/
│           ├── bronze/
│           │   ├── parcel_readings.csv
│           │   └── parcel_metadata.csv
│           │
│           ├── silver/
│           │   └── cleaned_parcel_timeseries.csv
│           │
│           └── gold/
│               └── crop_ndvi_analysis.csv
│
├── README.md
```




## 🏗️ Medallion Architecture (Bronze → Silver → Gold)

Raw CSV Data  
↓  
Cleaning & Standardization (Silver Layer)  
↓  
Validated Structured Outputs  
↓  
Analytics Layer (Gold - optional)

### 🥉 BRONZE LAYER (Raw Data)

Purpose: Store raw data exactly as received.

Input Files:
- parcel_readings.csv
- parcel_metadata.csv

Characteristics:
- No transformations
- No cleaning
- No validation
- Source-of-truth layer

---

### 🥈 SILVER LAYER (Cleaned Data)

Purpose: Clean, validate, and standardize raw data.

Output:
- result_cleaned_silver.csv

Transformations:
- Missing value handling (imputation / defaults)
- Duplicate removal (parcel_id + date)
- Date standardization (multiple formats → unified format)
- sensor_status normalization (ok/Ok/OK → OK)
- NDVI validation and clipping (-1 to 1)
- Categorical standardization (crop_type, etc.)
- Removal of corrupted records

---

### 🥇 GOLD LAYER (Business Analytics Layer)

Purpose: Generate business insights for decision-making.

Output:
- result_gold.csv

Schema:
| crop_type | mean_ndvi_before | mean_ndvi_after | n_parcels |

Business Logic:
- Compute NDVI average 30 days before sowing_date
- Compute NDVI average 30 days after sowing_date
- Consider only sensor_status = "OK"
- Aggregate results at crop level

---

## 📂 INPUT DATA

### parcel_readings.csv (Sensor Data)
- parcel_id → Unique parcel identifier
- date → Observation date (mixed formats)
- ndvi_value → Vegetation index (-1 to 1)
- temperature_c → Temperature in Celsius
- rainfall_mm → Rainfall in mm
- sensor_status → Sensor health flag

---

### parcel_metadata.csv (Master Data)
- parcel_id → Unique identifier
- mill_id → Mill identifier
- crop_type → Crop name (wheat, sugarcane, soybean, etc.)
- sowing_date → Crop sowing date
- area_hectares → Parcel size

---

## 🧪 DATA QUALITY ISSUES

# Possible Data Quality Issues

## Issue 1: Null or Missing Values

### Description
Some columns may contain:
- NULL values
- Empty strings
- Missing records

### Examples
- Missing `ndvi_value`
- Missing `temperature_c`
- Missing `sowing_date`

### Why This Is a Problem
Missing values can:
- Break calculations
- Cause incorrect averages
- Create join failures
- Produce invalid analytics

### Recommended Action

| Column Type | Action |
|---|---|
| Critical IDs (`parcel_id`) | Drop rows |
| Numerical columns | Impute if missing count is small |
| Important business columns | Flag or remove |

### Justification
Primary identifiers cannot be missing because joins and downstream analytics depend on them.

---

# Data Quality Audit Overview

A professional data quality audit typically checks the following:

| Check Type | Examples |
|---|---|
| Missing values | NULLs |
| Duplicate rows | Repeated records |
| Invalid ranges | NDVI outside `[-1,1]` |
| Invalid categories | Unknown sensor status |
| Bad dates | Future dates, invalid formats |
| Join integrity | Missing parcel IDs across tables |
| Inconsistent formatting | Uppercase/lowercase issues |
| Outliers | Unrealistic temperature/rainfall |

---

## Issue 2: Duplicate Records

### Description
The same:
- `parcel_id`
- `date`

may appear multiple times.

### Why This Is a Problem
Duplicates can lead to:
- Double counting
- Incorrect averages
- Invalid trend analysis
- Inflated reporting metrics

### Business Rule
Each parcel should have only one reading per date.

### Recommended Action
- Remove duplicate records using:
  - `parcel_id`
  - `date`
- Keep only the first valid occurrence.

### Justification
Duplicate records distort agricultural analytics and reduce data reliability.

---

## Issue 3: Invalid Dates

### Description
Date columns may:
- Fail parsing
- Contain future dates
- Use inconsistent formats

### Why This Is a Problem
Invalid dates can:
- Break time-series analysis
- Cause incorrect aggregations
- Create inaccurate seasonal trends

### Recommended Action
- Standardize all date formats
- Remove rows with invalid dates
- Reject future dates if not business-valid

### Justification
Accurate date formatting is critical for temporal agricultural analysis.

---

## Issue 4: Invalid NDVI Values

### Description
NDVI values must remain within the scientifically valid range:

- `-1 <= NDVI <= 1`

### Why This Is a Problem
Values outside this range indicate:
- Faulty satellite readings
- Corrupted sensor data
- Data ingestion issues

### Examples
- `1.5`
- `-2`

### Recommended Action
- Flag invalid records
- Replace invalid values with NULL
- Optionally remove affected rows

### Justification
NDVI is scientifically bounded between `-1` and `1`, making out-of-range values invalid for analysis.

---

## Issue 5: Invalid Temperature Values

### Description
Extreme temperature values may indicate faulty sensors.

### Examples
- `-100°C`
- `200°C`

### Recommended Valid Range
- `-20°C to 60°C`

### Why This Is a Problem
Unrealistic temperatures can:
- Distort crop health analysis
- Affect anomaly detection
- Produce unreliable dashboards

### Recommended Action
- Flag unrealistic values
- Replace with NULL or remove records

### Justification
Agricultural temperature readings generally remain within practical environmental limits.

---

## Issue 6: Negative Rainfall Values

### Description
Rainfall values cannot be negative.

### Why This Is a Problem
Negative rainfall indicates:
- Sensor malfunction
- Incorrect data ingestion
- Corrupted measurements

### Recommended Action
- Convert negative values to NULL
- Optionally remove affected rows

### Justification
Rainfall measurements are physically non-negative.

---

## Issue 7: Invalid Sensor Status Values

### Description
Sensor status values may contain inconsistent categories such as:
- `bad`
- `BAD`
- `faulty`
- `null`
- `unknown`

### Why This Is a Problem
Inconsistent categorical values can:
- Break filtering logic
- Produce incorrect aggregations
- Reduce reporting consistency

### Recommended Action
- Standardize values using:
  - lowercase conversion
  - whitespace trimming
- Allow only approved categories:
  - `good`
  - `bad`

### Justification
Consistent categorical formatting improves data reliability and analytics quality.

---

## Issue 8: Metadata Join Mismatch

### Description
Some `parcel_id` values may exist:
- In readings data but not metadata
- In metadata but not readings

### Why This Is a Problem
Join mismatches can:
- Create incomplete records
- Cause reporting inconsistencies
- Produce missing analytical context

### Recommended Action
- Identify unmatched records
- Flag missing parcel references
- Investigate source system inconsistencies

### Justification
Join integrity is essential for accurate parcel-level agricultural analytics.

---

## Issue 9: Invalid Area Hectares

### Description
Parcel area values cannot be:
- Zero
- Negative

### Why This Is a Problem
Invalid land area values affect:
- Yield calculations
- Productivity metrics
- Farm-level analytics

### Recommended Action
- Remove invalid rows
- Enforce positive area constraints

### Justification
Agricultural land area must always be greater than zero for meaningful analysis.

### parcel_readings.csv
- Mixed date formats (16/05/2026, 1/27/2026, 20-Jan-26)
- NDVI out of range (-1 to 1)
- Missing sensor_status values
- Null temperature and rainfall values
- Duplicate (parcel_id + date)
- Inconsistent casing in sensor_status

### parcel_metadata.csv
- Missing sowing_date values
- Duplicate parcel_id records
- Null/zero area_hectares
- Inconsistent crop_type formatting

### Cleaning Decisions

- NDVI values clipped to [-1, 1]
- Missing numeric values imputed or filled
- sensor_status standardized to uppercase
- Duplicate records removed using dropDuplicates
- Date formats standardized using Spark functions
- Missing metadata handled using imputation or retention

---
## 🧹 Data Pipeline (PySpark ETL Process)

### Steps

1. Data Ingestion
   - Load CSV files into Spark DataFrames
   - Schema inference enabled

2. Data Cleaning
   - Standardize formats
   - Normalize categorical fields
   - Handle missing values

3. Validation
   - Remove NDVI outliers
   - Filter invalid sensor_status records
   - Remove duplicates

4. Transformation
   - Prepare structured datasets
   - Align time-series data


## 🧹 SILVER LAYER PIPELINE

Steps:
1. Load raw CSV files into Spark DataFrames
2. Standardize schema and formats
3. Handle missing values (imputation / defaults)
4. Normalize categorical columns
5. Remove duplicates
6. Validate NDVI range [-1, 1]
7. Clean sensor_status values
8. Write cleaned dataset

Output:
result_cleaned_silver.csv

---

## 📊 NDVI Analysis (Crop Level)

### Objective

For each crop_type:
- Mean NDVI 30 days before sowing_date
- Mean NDVI 30 days after sowing_date
- Only include sensor_status = "OK"

---

## 🧠 GOLD LAYER PIPELINE

Steps:
1. Filter only sensor_status = "OK"
2. Join cleaned datasets
3. Compute time window:
   - 30 days before sowing_date
   - 30 days after sowing_date
4. Aggregate by crop_type
5. Compute:
   - mean_ndvi_before
   - mean_ndvi_after
   - n_parcels

Output:
result_gold.csv

---

## ⚙️ PIPELINE FLOW

parcel_readings.csv + parcel_metadata.csv  
        ↓  
🥉 BRONZE LAYER  
        ↓  
🥈 SILVER LAYER → result_cleaned_silver.csv  
        ↓  
🥇 GOLD LAYER → result_gold.csv  

---

## 🏭 PRODUCTION READINESS

If scaled to 100x data:

### Scalability
- Use Delta Lake / Parquet instead of CSV
- Partition by parcel_id and date
- Optimize Spark joins (broadcasting)

### Orchestration
- Databricks Workflows / Airflow
- Scheduled daily runs
- Retry + failure alerts

### Monitoring
- Missing NDVI percentage
- Sensor failure rate
- Schema drift detection
- Pipeline latency tracking

---

## ⚠️ MOST CRITICAL RISK

Sensor_status inconsistencies or missing values

Impact:
- Pipeline runs successfully
- But produces incorrect analytics silently

---

## 🤖 AI ASSISTANCE

This project was built with AI assistance (ChatGPT).

Used for:
- Architecture design (Medallion model)
- PySpark pipeline structuring
- Data cleaning strategy
- Documentation writing

All final code decisions were manually validated.

---

## ▶️ HOW TO RUN

git clone https://github.com/Mahesh6221/satellite_intelligence_assignment  
cd satellite_intelligence_assignment  

spark-submit data_pipeline.py  

---

## 📌 KEY LEARNINGS

- Real-world data is messy and requires strong cleaning
- Medallion architecture improves scalability and clarity
- Silver layer enables reusable datasets
- Gold layer focuses on business insights
- Data quality is more important than complexity
- Silent failures are the biggest production risk

---

## 🎥 LOOM VIDEO

https://loom.com/your-video-link

---

## 👤 AUTHOR

Mahesh Patil  
Data Engineer | Python | PySpark | Databricks  
GitHub: https://github.com/Mahesh6221