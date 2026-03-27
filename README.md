# 🏥 National Healthcare & Social Impact Analytics Platform

![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)
![MySQL](https://img.shields.io/badge/MySQL-8.0-orange.svg)
![Streamlit](https://img.shields.io/badge/Streamlit-Frontend-red.svg)
![Data Engineering](https://img.shields.io/badge/Data-Engineering-brightgreen.svg)

> **🔒 Note to Recruiters & Hiring Managers:** To maintain the integrity of the data pipelines and prevent unauthorized reuse, the raw source code, complex ETL scripts, and database dumps for this project are currently hosted in a **private repository**.
>
> This README serves as a comprehensive architectural report demonstrating the system design, data engineering decisions, and analytical capabilities of the platform. **Live demo and source code access are available upon request.**

## 📑 Executive Summary

This project is a national-scale data engineering and analytics platform that investigates the relationship between **healthcare resource allocation** and **social vulnerability**. By ingesting and normalizing over 50,000 raw records from disparate federal agencies (CMS, CDC, HealthData.gov), the system provides interactive, low-latency insights into hospital capacity, medical quality, and regional resource disparities.

## 🛠️ Tech Stack

* **Database Engine:** MySQL 8.0 (BCNF Normalization, Complex Joins, Window Functions)
* **ETL Pipeline:** SQL-driven In-Database Processing, Fuzzy String Matching
* **Frontend Dashboard:** Python, Streamlit, Pandas
* **Security & State:** SHA-256 Hashing for authentication, Session State Management

---

## 🏗️ System Architecture & Data Modeling

*[Screenshot Placeholder: Insert your 5-table E-R schema diagram here]*

### The Challenge: BCNF & Geographic Ambiguity
The initial flat CSV files presented severe functional dependency issues and update anomalies. A naive approach of using `ZIP Code` as a primary key fails in the real world because a single US ZIP code can span multiple cities or counties.

### The Solution: 5-Table Star Schema with Surrogate Keys
To achieve **Boyce-Codd Normal Form (BCNF)** and resolve geographical anomalies, I architected a robust 5-table schema:

| Table Name | Type | Source | Role in System | 
| ----- | ----- | ----- | ----- | 
| `Locations` | Dimension | CMS | Acts as a geographical encyclopedia. Generates a unique `location_id`. | 
| `Hospitals` | Fact / Bridge | **CMS (Raw)** | Core entity mapping facilities to `location_id`. | 
| `Hospital_Capacity` | Fact | HealthData | Tracks weekly ICU and bed utilization. | 
| `Hospital_Quality` | Fact | CMS | Tracks medical metrics (e.g., mortality, complications). | 
| `County_SVI` | Dimension | CDC | Social Vulnerability Index (Socioeconomic status). | 

---

## 🚀 Key Engineering Highlights

### 1. In-Database ETL & The "ZIP Code" Trap
Instead of pre-cleaning data in Excel, I implemented an **In-Database ETL pipeline**. To resolve the ZIP code ambiguity, I imported the *raw* hospital data into a staging format and utilized a three-factor composite match to assign the Surrogate Key (`location_id`):

```sql
-- Engineering Highlight: Resolving Geographic Ambiguity
UPDATE Hospitals h
JOIN Locations l ON 
    h.temp_zip = l.zip_code AND 
    h.temp_city = l.city AND 
    h.temp_state = l.state
SET h.location_id = l.location_id;
```

### 2. Fuzzy String Matching for Cross-Agency Data
Merging CMS medical data with CDC social data presented severe string inconsistencies (e.g., `"HOUSTON"` vs `"HOUSTON COUNTY"`). I engineered a normalized fuzzy join to prevent data loss:

```sql
-- Normalizing and fuzzy matching disparate federal datasets
JOIN County_SVI s ON l.state = s.state_abbr 
     AND UPPER(TRIM(s.county)) LIKE CONCAT(UPPER(TRIM(l.county)), '%')
```
*Result: Successfully linked >95% of national hospitals to their respective socioeconomic profiles.*

---

## 📊 Interactive Analytics & Visualizations

The platform features a secure, Streamlit-based Single Page Application (SPA) that translates complex SQL queries into intuitive visual dashboards. 

*[Screenshot Placeholder: Insert a wide screenshot of your Streamlit Dashboard homepage/login screen here]*

The system includes **10 predefined analytical modes** optimized for performance, categorized into four core domains:

### 🌍 1. National Capacity & Resource Distribution
Visualizing macro-level infrastructure scale and disparities.
* **Top 20 Largest Hospitals (National):** Identifies macro-scale medical centers via `ORDER BY` capacity.
* **Hospital Counts by State:** A comparative bar chart highlighting medical infrastructure density.
* **National Bed Capacity Trend (Weekly):** A time-series line chart tracking temporal resource fluctuations across all US hospitals.
  
*[Screenshot Placeholder: Insert a screenshot of the "National Bed Capacity Trend" Line Chart here]*

### ⚖️ 2. Social Vulnerability (SVI) Impact
Bridging the gap between socioeconomic status and healthcare access.
* **Top 10 Most Vulnerable States (Avg SVI):** Aggregates CDC data to highlight states with the highest systemic risk, visualized through high-contrast bar charts.

*[Screenshot Placeholder: Insert a screenshot of the "Top 10 Most Vulnerable States" Bar Chart here]*

### 🏥 3. Medical Quality & ICU Intensity
* **ICU Intensity Ranking (Top 15):** Calculates derived metrics (`icu_avg / beds_avg`) to identify hospitals operating under critical intensive care loads.
* **Hospital Ratings Distribution:** Analyzes the average CMS star ratings across the top 10 states.
* **Average Score by Medical Measure:** Evaluates national performance across specific clinical metrics (e.g., mortality rates).

*[Screenshot Placeholder: Insert a screenshot of the "ICU Intensity Ranking" or "Quality" chart here]*

### 📍 4. Regional Deep-Dives
Executing highly filtered queries for targeted geographic analysis.
* **DMV Area Deep Dive:** Grouped bar charts categorizing hospital types specifically in DC, MD, and VA.
* **New York City (NY) Capacity Explorer:** Granular capacity ranking for the NY metropolitan area.
* **High-Scale Providers in Florida (FL):** Aggregated capacity sum by hospital type in FL.

*[Screenshot Placeholder: Insert a screenshot of the "DMV Area Deep Dive" Grouped Bar Chart here]*

---
*Architected and developed as an end-to-end Data Engineering & Database Systems project.*
