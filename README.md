🏥 National Healthcare & Social Impact Analytics Platform
🔒 Note to Recruiters & Hiring Managers: To maintain the integrity of the data pipelines and prevent unauthorized reuse, the raw source code, complex ETL scripts, and database dumps for this project are currently hosted in a private repository.
This README serves as a comprehensive architectural report demonstrating the system design, data engineering decisions, and technical challenges solved during this project. Live demo and source code access are available upon request during interviews.
📑 Executive Summary
This project is a national-scale data engineering and analytics platform that investigates the relationship between healthcare resource allocation and social vulnerability. By ingesting and normalizing over 50,000 raw records from disparate federal agencies (CMS, CDC, HealthData.gov), the system identifies "medical deserts"—highly vulnerable communities facing critical hospital capacity shortages.
🛠️ Tech Stack
Database: MySQL 8.0 (BCNF Normalization, Complex Joins, Window Functions)
ETL Pipeline: SQL-driven In-Database Processing, Data Imputation
Frontend: Python, Streamlit, Pandas
Security: SHA-256 Hashing, Session State Management
🏗️ System Architecture & Data Modeling
The Challenge: BCNF & Geographic Ambiguity
The initial flat CSV files presented severe functional dependency issues and update anomalies. A naive approach of using ZIP Code as a primary key fails in the real world because a single US ZIP code can span multiple cities or counties.
The Solution: 5-Table Star Schema with Surrogate Keys
To achieve Boyce-Codd Normal Form (BCNF) and resolve geographical anomalies, I architected a robust 5-table schema:
Table Name
Type
Source
Role in System
Locations
Dimension
CMS
Acts as a geographical encyclopedia. Generates a unique location_id.
Hospitals
Fact / Bridge
CMS (Raw)
Core entity mapping facilities to location_id.
Hospital_Capacity
Fact
HealthData
Tracks weekly ICU and bed utilization.
Hospital_Quality
Fact
CMS
Tracks medical metrics (e.g., mortality, complications).
County_SVI
Dimension
CDC
Social Vulnerability Index (Socioeconomic status).

🚀 Key Engineering Highlights
1. In-Database ETL & The "ZIP Code" Trap
Instead of pre-cleaning data in Excel, I implemented an In-Database ETL pipeline. To resolve the ZIP code ambiguity, I imported the raw hospital data into a staging format and utilized a three-factor composite match to assign the Surrogate Key (location_id):
-- Engineering Highlight: Resolving Geographic Ambiguity
UPDATE Hospitals h
JOIN Locations l ON 
    h.temp_zip = l.zip_code AND 
    h.temp_city = l.city AND 
    h.temp_state = l.state
SET h.location_id = l.location_id;


This approach guaranteed 100% accurate geographical mapping by leveraging the database's indexing capabilities, later dropping the temporary columns to maintain schema purity.
2. Fuzzy String Matching for Cross-Agency Data
Merging CMS medical data with CDC social data presented severe string inconsistencies (e.g., "HOUSTON" vs "HOUSTON COUNTY"). I engineered a normalized fuzzy join to prevent data loss:
-- Normalizing and fuzzy matching disparate federal datasets
JOIN County_SVI s ON l.state = s.state_abbr 
     AND UPPER(TRIM(s.county)) LIKE CONCAT(UPPER(TRIM(l.county)), '%')


Result: Successfully linked >95% of national hospitals to their respective socioeconomic profiles without manual intervention.
3. Data Integrity & Anomaly Handling
Handled severe anomalies in federal time-series data, specifically converting missing values explicitly represented as -999,999 into SQL NULL values during the LOAD DATA INFILE process to prevent heavily skewed AVG() aggregations.
📊 Interactive Analytics Dashboard
To make the data accessible, I developed a Single Page Application (SPA) using Streamlit:
Secure Access: Implemented a stateless login mechanism utilizing SHA-256 hashing (hashlib) to protect the dashboard, demonstrating an understanding of secure credential storage (Pre-image Resistance).
High-Performance Querying: Dashboard modes (e.g., Top 10 Most Vulnerable States, DMV Area Deep Dive) are powered by highly optimized SQL queries utilizing aggregations and RANK() OVER (PARTITION BY...) window functions, returning results in milliseconds.
Dynamic Visualization: Automatically renders Scatter plots, Bar charts, and Line graphs based on the selected analytical dimension.
Architected and developed by Derry as an end-to-end Data Engineering & Database Systems project.
