# satellite_intelligence_assignment

# 🌍 Satellite Intelligence – Data Engineering Pipeline (Databricks + PySpark)

## 🚀 Project Overview

This project is a production-style data engineering pipeline built using PySpark on Databricks, designed to simulate real-world agricultural satellite intelligence workflows.

It processes messy agricultural field data (sensor readings + parcel metadata) and converts it into clean, structured datasets for analytics.

In real-world systems, agricultural data is:
- Missing and incomplete
- Noisy and inconsistent
- Sensor-failure prone
- Time-series dependent

This pipeline demonstrates how to handle such data using scalable Spark-based architecture.

---

## 📂 Input Data Sources

### parcel_readings.csv
Daily sensor-level readings per parcel:

- parcel_id → Unique field identifier  
- date → Observation date (multiple formats)  
- ndvi_value → Vegetation index (-1 to 1)  
- temperature_c → Daily temperature  
- rainfall_mm → Daily rainfall  
- sensor_status → Sensor health flag  

---

### parcel_metadata.csv
Static parcel master data:

- parcel_id → Unique field identifier  
- mill_id → Sugar mill identifier  
- crop_type → Crop category  
- sowing_date → Crop sowing date  
- area_hectares → Parcel size  

---

## ⚙️ Architecture (Medallion Pipeline)

Bronze → Silver → Gold Architecture:

Raw CSV Data  
↓  
Cleaning & Standardization (Silver Layer)  
↓  
Validated Structured Outputs  
↓  
Analytics Layer (Gold - optional)

---

## 🧪 Data Quality Audit

### Issues in parcel_readings.csv
- Mixed date formats (16/05/2026, 1/27/2026, 20-Jan-26)
- NDVI values outside valid range [-1, 1]
- Missing sensor_status values
- Null temperature and rainfall values
- Duplicate records (parcel_id + date)
- Inconsistent casing in sensor_status (ok / OK)

---

### Issues in parcel_metadata.csv
- Missing sowing_date values
- Duplicate parcel_id entries
- Null or zero area_hectares
- Inconsistent crop_type formatting

---

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

---

## 📤 Final Output Files

Instead of a single merged dataset, the pipeline produces two clean silver-layer datasets.

---

### 1. Cleaned Parcel Readings

result_parcel_readings.csv

Contains:
- Cleaned time-series sensor data
- Valid NDVI values
- Standardized sensor_status
- Duplicate-free records

---

### 2. Cleaned Parcel Metadata

result_parcel_metadata.csv

Contains:
- Cleaned parcel master data
- Standardized crop types
- Valid sowing dates
- Deduplicated records

---

### Pipeline Flow

parcel_readings.csv
↓
Cleaning + Validation
↓
result_parcel_readings.csv

parcel_metadata.csv
↓
Cleaning + Validation
↓
result_parcel_metadata.csv

---

## 🧠 Design Decision

Both datasets are cleaned separately instead of immediate joining.

Why:
- Better modular design
- Easier debugging
- Reusable datasets
- Aligns with Medallion architecture (Silver layer design)

---

## 📊 NDVI Analysis (Crop Level)

### Objective

For each crop_type:
- Mean NDVI 30 days before sowing_date
- Mean NDVI 30 days after sowing_date
- Only include sensor_status = "OK"

---

### Output

| crop_type | mean_ndvi_before | mean_ndvi_after | n_parcels |
|-----------|------------------|-----------------|------------|
| sugarcane | X.XX             | X.XX            | XXX        |
| wheat     | X.XX             | X.XX            | XXX        |
| soybean   | X.XX             | X.XX            | XXX        |

---

## 🏭 Production Readiness (100x Scale)

### 1. Scalability
- Use Delta Lake / Parquet instead of CSV
- Partition by parcel_id and date
- Optimize Spark joins

### 2. Orchestration
- Use Databricks Workflows / Airflow
- Schedule incremental pipelines
- Add retry + alerting

### 3. Monitoring
- Missing NDVI rate
- Sensor failure rate
- Schema drift detection
- Data freshness monitoring

---

### ⚠️ Most Likely Failure

Sensor_status inconsistencies or missing values

Impact:
- No pipeline failure
- But incorrect analytics results

---

## 🤖 AI Tools Used

This project was developed using AI assistance (ChatGPT).

Used for:
- Pipeline design guidance
- ETL architecture structuring
- Data cleaning strategy
- Documentation writing

All final code decisions were validated manually.

---

## ▶️ How to Run

### Clone Repository
git clone https://github.com/Mahesh6221/satellite_intelligence_assignment
cd satellite_intelligence_assignment

### Run in Databricks
- Upload CSVs to DBFS or Volume storage
- Run notebook or Python script

### Run Pipeline
spark-submit data_pipeline.py

---

## 📌 Key Learnings

- Real-world data is messy and requires heavy cleaning
- Time-series joins are sensitive to data quality
- Spark is essential for scalable pipelines
- Data quality > model complexity
- Silent data corruption is the biggest production risk

---

## 🎥 Loom Video

https://loom.com/your-video-link

---

## 👤 Author

Mahesh Patil  
Data Engineer | Python | PySpark | Databricks  
GitHub: https://github.com/Mahesh6221