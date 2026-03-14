# Erdős Project: Predicting the SNAP Gap

## 1. Project Objective
The goal of this project is to identify the socioeconomic and demographic barriers that prevent eligible low-income households from receiving Supplemental Nutrition Assistance Program (SNAP) benefits. 

Instead of looking at aggregate county-level data, we are modeling this "SNAP Gap" directly at the **household level** using Census microdata. By framing this as a classification problem, we aim to extract feature importance and coefficients to understand *why* people fall through the safety net (e.g., technology barriers, language isolation, transportation access).

## 2. Data Strategy & Target Definition
**Data Source:** 2023 American Community Survey (ACS) 1-Year Public Use Microdata Sample (PUMS) for MD, DC, and VA. We utilize both the Housing (`h`) and Person (`p`) datasets.

**Defining the Target Population:**
To prevent the model from learning the trivial rule that "higher income equals no SNAP," we restrict our training data strictly to the universally eligible low-income cohort.
* **Filter:** Gross household income must be at or below 130% of the poverty line (`POVPIP <= 130`). 
* *Note on EDA findings:* We identified a sub-population legally claiming SNAP with gross incomes > 130% (due to Broad-Based Categorical Eligibility and net-income deductions). While important for policy context, they are excluded from the machine learning model to maintain a clean target variable.
* **Filter:** Exclude Group Quarters (GQ) to focus on standard households.
* **Filter:** Exclude rows with Census-imputed SNAP values (`FFSP == 1`) to ensure the model trains only on self-reported ground truth.

**Defining the Target Variable (`y`):**
* **Class 1 (Claiming):** Eligible (`POVPIP <= 130`) AND receives SNAP (`FS == 1`).
* **Class 0 (The Gap):** Eligible (`POVPIP <= 130`) BUT does not receive SNAP (`FS == 2`).

## 3. Data Processing & Feature Engineering (Completed)
Since SNAP is awarded at the household level, but many barriers are individually experienced, we aggregated person-level demographics into household-level summary features using `SERIALNO` as the merge key.

**Engineered Barrier Features:**
* **Vulnerability Flags:** `HAS_ELDERLY` (60+), `HAS_DISABLED`
* **Household Composition:** `NUM_CHILDREN`, `NUM_WORKING_ADULTS`, `NP` (Household size)
* **Demographics:** `MAX_EDUCATION`, `IS_MINORITY_HH`
* **Access & Infrastructure:** `VEH` (Vehicles available), `ACCESSINET` (Internet access)
* **Language:** `LNGI` (Limited English speaking household)
* **Housing Stability:** `GRPIP` (Gross rent as % of income), `TEN` (Tenure/Ownership), `RMSP` (Number of rooms)
* **Financials:** `HINCP` (Household income), `POVPIP` (Income-to-poverty ratio)

Data bloat (e.g., over 160 Census replicate weight columns) was purged to prepare a lean feature matrix.

## 4. Current Phase: Baseline Modeling
With the processed dataset generated and documented, the immediate next step is to establish a baseline predictive model. 

**Plan for First Attempt:**
1.  **Data Splitting:** Split the clean low-income cohort into Train (70%), Validate (15%), and Test (15%) sets to ensure rigorous evaluation.
2.  **Scaling:** Apply `StandardScaler` to continuous and ordinal features (like Income and Education) so the model treats feature magnitudes appropriately.
3.  **Model Selection:** Train a minimal **Logistic Regression** model (with balanced class weights to account for any class imbalances).
4.  **Goal:** We are not just looking for predictive accuracy; we will examine the model's coefficients (and potentially switch to `statsmodels` for p-values) to identify which engineered features are statistically significant drivers of households falling into the SNAP Gap.