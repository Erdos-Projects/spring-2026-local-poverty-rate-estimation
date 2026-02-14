# Data Inventory & Schema

## 1. Unit of Analysis
* **Training Level:** Individual Person (ACS Public Use Microdata Sample) 
* **Prediction Level:** Census Tract (ACS Aggregate Estimates)
* **Support Level:** County (SAIPE & SNAP Administrative Records)

## 2. Cleaned Data Schema
This project relies on data at three geographic resolutions plus a linkage table. Below are the definitive columns used for modeling.

### A. The "Canvas": Tract Targets (`df_tract_enriched`)
**Source:** Census ACS 5-Year Estimates (2023) via Census API. It is a 5-year moving aggregation (2018-2023).
**Role:** The demographic buckets we apply the model to (The "Small Areas").

| Column Name | Type | Description |
| :--- | :--- | :--- |
| `GEOID` | String (11) | Unique Tract ID (State+County+Tract). Primary Key. |
| `GEOID_COUNTY` | String (5) | Foreign Key. State+County FIPS (links to County Data). |
| `total_population` | Int | Denominator for all rates (B01003). |
| `poverty_count_est` | Int | Official ACS poverty count (Target for smoothing). |
| `poverty_count_moe` | Int | **Margin of Error** (90% CI). The uncertainty floor we aim to beat. |
| **Race Buckets** | Counts | 7 Columns: White, Black, Native, Asian, Pacific, Other, Two+. |
| **Age/Sex Buckets** | Counts | 46 Columns (Joint Distribution). See Split Points below. |

**Feature Definitions (The Buckets)**
* **Sex:** Binary (Male, Female)
* **Age Split Points (Lower Bounds):**
    * `[0, 5, 10, 15, 18, 20, 21, 22, 25, 30, 35, 40, 45, 50, 55, 60, 62, 65, 67, 70, 75, 80, 85]`
    * *Note:* These create 23 age bins per sex (e.g., `m_00_04` ... `m_85_plus`).

### B. The Training Data: Individuals (`df_person_enriched`)
**Source:** ACS PUMS Person File (Cleaned). 
**Role:** Microdata used to train the classifier (The "Behavior").

| Column Name | Type | Description |
| :--- | :--- | :--- |
| `PUMA` | String | Public Use Microdata Area. Rough geographic location (~100k people). |
| `is_poor` | Binary (0/1) | **Target Variable.** Derived from `POVPIP < 100`. |
| `Person_Weight` | Int | `PWGTP`. Expansion weight (people represented by this row). |
| `Age` | Int | 0-99. Mapped to the split points above. |
| `Sex_Code` | Int | 1=Male, 2=Female. Mapped to buckets above. |
| `Race_Code` | Int | Census code mapped to the 7 Race Buckets above. |

**Excluded Columns & Rationale:**
We intentionally discard high-value predictors found in the raw PUMS data—such as **Education Level (SCHL), Employment Status (ESR), Industry (INDP), and Housing Costs (RNTP)**. 
* **Reason:** The "Lowest Common Denominator" constraint. We can only train on variables that exist as **joint distributions** in the Tract-level data (Section A). Since the Census does not publish a "Age x Sex x Race x Education x Industry" table for every tract, we cannot use those features for prediction, even if they are strong predictors of poverty.

### C. Support Training Data: County Level (`df_county`)
**Source:** USDA SNAP Admin Records & Census SAIPE.
**Role:** High-confidence aggregate anchors used to calibrate or constrain the tract-level predictions.

| Column Name | Type | Description |
| :--- | :--- | :--- |
| `GEOID_COUNTY` | String (5) | FIPS Code (e.g., `24031`). Primary Key. |
| `snap_persons_total`| Int | **USDA SNAP:** Actual count of benefit recipients. |
| `saipe_poverty_est` | Int | **SAIPE:** Modeled poverty estimate (admin-adjusted). |
| `saipe_poverty_moe` | Int | Margin of error for the SAIPE estimate. |

### D. Geographic Bridge: PUMA-County Crosswalk (`df_puma_map`)
**Source:** IPUMS USA / US Census Bureau (2020 Relationship File).
**Role:** Solves the spatial mismatch between Training Units (PUMAs) and Prediction Units (Counties/Tracts).

| Column Name | Type | Description |
| :--- | :--- | :--- |
| `PUMA` | String | Unique PUMA ID (State+PUMA). Primary Key. |
| `tract_ids` | List[String] | List of all Census Tract GEOIDs contained within this PUMA. |
| `counties` | List[String] | List of all County FIPS codes intersecting this PUMA. |
| `weights` | List[Float] | **Population Weights.** Used to distribute PUMA-level effects to counties. (e.g., if a Rural PUMA is 30% County A and 70% County B). |