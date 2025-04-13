
# Objective  
This project seeks to derive actionable insights from the UK Road Safety Accident Data for the year 2023. By leveraging structured accident, vehicle, and casualty records published by the Department for Transport, the analysis aims to support data-informed decision-making in road safety management.

The primary objective is to identify critical patterns in road traffic incidents with a focus on spatial distribution, temporal trends, environmental conditions, demographic vulnerability, and vehicle involvement. These insights are intended to assist public authorities, urban planners, and policy stakeholders in optimizing intervention strategies and resource allocation.

## Key Performance Indicators (KPIs) 

| KPI / Variable | Description |
|----------------|-------------|
| **Total Reported Accidents** | Total number of recorded traffic accidents in 2023. |
| **Accident Severity Classification** | Categorization of accidents as Fatal, Serious, or Slight. |
| **Accident Frequency by Time** | Number of accidents segmented by hour of day, day of week, and month. |
| **Geographic Accident Distribution** | Spatial density of accidents based on latitude and longitude. |
| **Casualty Demographics** | Distribution of casualties by age group, gender, and injury severity. |
| **Environmental Impact on Accidents** | Accident counts under different weather, lighting, and road surface conditions. |
| **Vehicle Type Involvement** | Types of vehicles most frequently involved in high-severity accidents. |


## Research Questions

1. **Where do the most accidents occur geographically across the UK?**  

2. **When are accidents most likely to happen?**  

3. **What environmental factors contribute to higher accident severity?**  

4. **Which age groups or genders are more vulnerable in accidents?**  

5. **What types of vehicles are most often involved in fatal or serious accidents?**  

6. **How can accident trends inform road safety planning and emergency response?**  

# Data Source

The data used in this project was obtained from the UK Department for Transport’s official open data portal:

[UK Road Safety Data (2023)](https://data.gov.uk/dataset/cb7ae6f0-4be6-4937-998c-40c9b55b3d20/road-safety-data)

It includes three CSV files: `Accidents_2023.csv`, `Vehicles_2023.csv`, and `Casualties_2023.csv`.

# Design and Mockup

## Dashboard Mockup
The Power BI dashboard is designed as a **single-page layout** with the following key sections:

- **Filter Bar (Top):** Year, Region, Severity, Weather, Time of Day.
- **KPI Summary:** Total accidents, severity breakdown, total casualties.
- **Geographic Map:** Accident hotspots visualized by location.
- **Time Analysis:** Accidents by hour, day of week, and month.
- **Demographics:** Casualty distribution by age, gender, and severity.
- **Vehicle Analysis:** Vehicle types involved in fatal or serious accidents.
- **Environmental Factors:** Impact of weather, light, and road surface on severity.

The layout ensures a clear overview and interactive exploration of accident trends to support road safety planning.

## 🛠️ Tools

| Tool              | Purpose                                                                 |
|-------------------|-------------------------------------------------------------------------|
| **Microsoft Excel** | Used for initial data overview and structure inspection of raw CSVs.   |
| **SQL Server**     | Data import, cleaning, transformation, and relational modeling.         |
| **Power BI Desktop** | Dashboard creation with KPIs, maps, time trends, and interactive visuals. |
| **Power Query**     | In-app data shaping, relationship setup, and table integration.         | 

# Project Stages

1. **Data Exploration**  
   Performed a preliminary review of the raw CSV files to understand structure, detect anomalies, and plan the cleaning process.

2. **Data Import and Cleaning**  
   Imported the raw CSV files into SQL Server and performed cleaning tasks, including type conversion, null handling, and decoding categorical variables.

3. **Data Modeling**  
   Defined relationships between cleaned tables (`Accidents`, `Vehicles`, `Casualties`) in SQL Server and connected Power BI to the final model.

4. **Dashboard Design**  
   Built a single-page Power BI dashboard featuring KPIs, interactive filters, geographic mapping, time-series charts, demographic analysis, and environmental factor breakdowns.

5. **Documentation**  
   Created structured README documentation outlining objectives, KPIs, research questions, tools, project stages, and dashboard layout for portfolio presentation.

# Data Exploration

A preliminary review of the three datasets was conducted using Microsoft Excel to understand their structure, detect anomalies, and prepare for downstream cleaning in SQL Server.

*Accidents Dataset (`dft-road-casualty-statistics-collision-2023.csv`)*
**Rows:** 104,258 | **Columns:** 37 |
- **Primary Key:** `accident_index`
- **Key Fields for Analysis:** `accident_severity`, `date`, `time`, `weather_conditions`, `light_conditions`, `road_surface_conditions`, `latitude`, `longitude`
- **Anomalies Detected:**
  - 12 rows missing geospatial data (`latitude`, `longitude`)
  - Time stored as text, needs standardization
  - Several categorical columns stored as codes (e.g., `accident_severity = 1` → "Fatal")
- **Cleaning Plan:**
  - Remove or flag rows with missing coordinates (for map visuals)
  - Convert `time` to valid `TIME` format
  - Decode all categorical codes using official lookup values

*Vehicles Dataset (`dft-road-casualty-statistics-vehicle-2023.csv`)*
**Rows:** 189,815 | **Columns:** 34 |
- **Primary Key:** `accident_index` + `vehicle_reference`
- **Key Fields for Analysis:** `vehicle_type`, `vehicle_manoeuvre`, `age_of_driver`, `sex_of_driver`, `journey_purpose_of_driver`
- **Anomalies Detected:**
  - 170,000+ rows missing directional vector columns (`dir_from_e/n`, `dir_to_e/n`) — not required for this project
  - Some fields (e.g., `age_of_driver`) contain outliers (e.g., extreme values or placeholders like `999`)
- **Cleaning Plan:**
  - Drop unused directional vector columns
  - Handle or remove outlier ages using value thresholds
  - Decode categorical variables using official mappings

*Casualties Dataset (`dft-road-casualty-statistics-casualty-2023.csv`)*
**Rows:** 132,977 | **Columns:** 21 |
- **Primary Key:** `accident_index` + `casualty_reference`
- **Key Fields for Analysis:** `age_of_casualty`, `sex_of_casualty`, `casualty_severity`, `casualty_type`, `casualty_class`
- **Anomalies Detected:**
  - Age values span a wide range; may include placeholders
  - All fields are populated, but categorical codes must be decoded
- **Cleaning Plan:**
  - Validate and group age values (e.g., into age bands)
  - Decode categorical fields (`casualty_severity`, `casualty_type`, etc.)

#### General Cleaning Strategy
- Use `accident_index` as the primary join key across all tables
- Standardize data types for date, time, and codes
- Decode all relevant categorical fields using reference mappings
- Prepare clean, relational data views for loading into Power BI

# Data Import and Cleaning 

- Imported all raw CSVs as `raw_*` tables with `VARCHAR(MAX)` types
- Created 15 lookup tables to decode categorical fields using STATS20
- Cleaned, cast, and validated all relevant fields (date, time, numeric)
- Filtered invalid rows (e.g., missing coordinates, invalid ages)
- Built cleaned views:
  - `vw_clean_accidents`
  - `vw_clean_vehicles`
  - `vw_clean_casualties`

## 🧩 Example: SQL View – `vw_clean_accidents`

```sql
CREATE VIEW vw_clean_accidents AS
SELECT
    accident_index,
    CAST(date AS DATE) AS accident_date,
    TRY_CAST(time AS TIME) AS accident_time,
    latitude,
    longitude,
    s.description AS accident_severity
FROM raw_accidents
LEFT JOIN lookup_accident_severity s
    ON TRY_CAST(accident_severity AS INT) = s.code
WHERE latitude IS NOT NULL AND longitude IS NOT NULL;
```

For full SQL view definitions, see the [`/sql/`](./sql/) folder.
