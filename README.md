EDA on Medicare Spending Per Beneficiary (MSPB)

# Exploratory Data Analysis of U.S. hospital efficiency using Medicare Spending Per Beneficiary metrics.

🔍 Project Overview

This repository demonstrates a complete end-to-end EDA workflow using SQL and Power BI to analyze how much Medicare spends per patient episode at individual hospitals compared to the national median (MSPB = 1.0).

Key goals:

Identify geographic and institutional spending patterns

Quantify high- and low-cost outliers

Showcase data cleaning, aggregation, and visualization best practices

📁 Repository Structure

EDA-MSPB/
├── SQL-code.sql            # All data-cleaning & EDA queries (no comments)
├── MSPB-Dataset.csv        # Raw CMS MSPB data (2020) — *download separately*
├── Power-BI-Report.pdf     # Final interactive dashboard snapshots (exported)
└── README.md               # Project overview, setup, queries, visuals

🛠 Data Source

CMS Provider Data – “Medicare Spending Per Beneficiary”

Source: https://data.cms.gov/provider-data/dataset/rs6n-9qwg

Note: CMS applies price standardization and risk adjustment to remove geographic and patient-health differences. Use the official dataset for accurate, up-to-date fields and metadata.

⚙️ Setup & Usage

1. Clone repository

git clone https://github.com/YOUR_USERNAME/EDA-MSPB.git
cd EDA-MSPB

2. Download dataset

Download MSPB-Dataset.csv from the CMS link above and place it into the repo root.

3. Import data into MySQL (example)

CREATE DATABASE eda_mspb;
USE eda_mspb;

-- Adjust path to where MSPB-Dataset.csv is stored on the DB host
LOAD DATA INFILE 'MSPB-Dataset.csv'
INTO TABLE mspb
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
IGNORE 1 ROWS;

If you prefer SQLite or Postgres, adapt the import commands accordingly.

4. Run SQL queries

Open SQL-code.sql in your SQL editor and execute sequentially to clean, profile, and analyze the dataset.

5. Visualize in Power BI

Open Power BI Desktop.

Connect to MySQL (or import CSV).

Import cleaned dataset or use SQL views as source.

Suggested visuals:

State-wise hospital counts (bar chart)

MSPB ratio distribution (<1.0, =1.0, >1.0) (pie / donut)

Top 10 highest / lowest MSPB hospitals (table)

Choropleth / filled map of average MSPB by state

Scatter: MSPB vs. number of discharges / episodes

📊 Key SQL Queries

Below are 10 practical SQL queries for cleaning and exploratory analysis. Put these (without comments) into SQL-code.sql.

SELECT COUNT(DISTINCT Provider_ID) AS total_hospitals,
       COUNT(DISTINCT State) AS total_states
FROM mspb
WHERE Score <> 'Not Available';

SELECT Provider_ID, Provider_Name, State, CAST(Score AS DECIMAL(6,3)) AS MSPB
FROM mspb
WHERE Score <> 'Not Available'
ORDER BY MSPB DESC
LIMIT 10;

SELECT Provider_ID, Provider_Name, State, CAST(Score AS DECIMAL(6,3)) AS MSPB
FROM mspb
WHERE Score <> 'Not Available'
ORDER BY MSPB ASC
LIMIT 10;

SELECT State, COUNT(*) AS hospital_count, AVG(CAST(Score AS DECIMAL(6,3))) AS avg_mspb
FROM mspb
WHERE Score <> 'Not Available'
GROUP BY State
ORDER BY hospital_count DESC;

SELECT CASE
         WHEN CAST(Score AS DECIMAL(6,3)) < 1.0 THEN '<1.0'
         WHEN CAST(Score AS DECIMAL(6,3)) = 1.0 THEN '=1.0'
         ELSE '>1.0'
       END AS mspb_category,
       COUNT(*) AS hospitals
FROM mspb
WHERE Score <> 'Not Available'
GROUP BY mspb_category;

SELECT COUNT(DISTINCT Provider_ID) AS count_eq_1
FROM mspb
WHERE Score = '1' OR Score = '1.0';

SELECT Provider_ID, Provider_Name, State, County, City, CAST(Score AS DECIMAL(6,3)) AS MSPB
FROM mspb
WHERE Score <> 'Not Available' AND CAST(Score AS DECIMAL(6,3)) > 2.0
ORDER BY MSPB DESC;

SELECT State, MIN(CAST(Score AS DECIMAL(6,3))) AS min_mspb,
       MAX(CAST(Score AS DECIMAL(6,3))) AS max_mspb,
       AVG(CAST(Score AS DECIMAL(6,3))) AS avg_mspb
FROM mspb
WHERE Score <> 'Not Available'
GROUP BY State
ORDER BY avg_mspb DESC;

SELECT Provider_ID, Provider_Name, CAST(Score AS DECIMAL(6,3)) AS MSPB, Total_MSPB_Episodes
FROM mspb
WHERE Score <> 'Not Available' AND Total_MSPB_Episodes IS NOT NULL
ORDER BY Total_MSPB_Episodes DESC
LIMIT 20;

SELECT COUNT(*) AS null_count, SUM(CASE WHEN Score = 'Not Available' THEN 1 ELSE 0 END) AS not_available_count
FROM mspb;

Note: Column names (e.g., Score, Provider_ID, Provider_Name, Total_MSPB_Episodes) may differ depending on the CMS CSV schema version — adapt the queries to match exact column names in your file.

📈 Power BI Dashboard Highlights

State-wise Hospital Count — bar chart showing counts per state

MSPB Ratio Breakdown — pie chart for <1.0, =1.0, >1.0

Top/Bottom Performers — table with hospital names and MSPB scores

Map Visualization — state-level average MSPB or county-level where available

Include exported snapshots in Power-BI-Report.pdf for quick preview and documentation.

🌟 Insights & Impact (example findings)

High Efficiency: ~35% of hospitals have MSPB < 1.0 (example stat — compute on your dataset)

Cost Outliers: Identified top 10 high-spending hospitals for deeper review

Regional Patterns: Some states show systematically higher/lower MSPB averages

🤝 Contributing

Fork this repo

Create a feature branch: git checkout -b feature/visual

Commit your changes: git commit -m "Add new visual"

Push: git push origin feature/visual

Open a Pull Request
