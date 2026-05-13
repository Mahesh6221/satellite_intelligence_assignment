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