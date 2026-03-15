# Erdős Project: Predicting the SNAP Gap via Socioeconomic Proxies

## 1. Project Objective
Build a predictive model to identify vulnerable households eligible for SNAP but not receiving it (the "SNAP Gap"). We use observable socioeconomic and demographic proxy features (e.g., household size, education, language barriers, internet access) to classify these households, enabling pure prediction for outreach tools. 

## 2. Data Strategy & Target Definition
**Data Source:** 2023 American Community Survey (ACS) 1-Year Public Use Microdata Sample (PUMS) for MD, DC, and VA. We use both the Housing (`h`) and Person (`p`) datasets.

**Defining the Target Variable (`y`):**
We use the entire standard population and construct our target variable using underlying income and SNAP data. 
* **Class 1 (The Gap):** Household is low-income (`POVPIP <= 130`) AND does not receive SNAP (`FS == 2`).
* **Class 0 (Not in the Gap):** All other households.

*Filters:* We exclude Group Quarters (GQ) and households with Census-imputed SNAP values (`FFSP == 1`) to ensure ground-truth accuracy. We drop the explicit income and SNAP columns from the feature matrix before training the model.

## 3. Data Processing & Feature Engineering
We aggregate person-level data into household-level summary features using `SERIALNO` as the merge key.

**Poverty Ratio (`POVPIP`):** The Census Bureau stores the poverty ratio in the Person dataset. We aggregate this to the household level by taking the `.first()` record for each `SERIALNO` group.

**Predictive Proxy Features (`X`):**
* **Vulnerability Flags:** `HAS_ELDERLY` (60+), `HAS_DISABLED`
* **Household Composition:** `NUM_CHILDREN`, `NUM_WORKING_ADULTS`, `NP` (Household size)
* **Demographics:** `MAX_EDUCATION`, `IS_MINORITY_HH`
* **Access & Infrastructure:** `VEH` (Vehicles available), `ACCESSINET` (Internet access)
* **Language:** `LNGI` (Limited English speaking household)
* **Housing Stability:** `GRPIP` (Gross rent as % of income), `TEN` (Tenure/Ownership), `RMSP` (Number of rooms)

## 4. Baseline Modeling Plan
1. **Data Splitting:** Split the population into Train (70%), Validate (15%), and Test (15%) sets.
2. **Scaling:** Apply `StandardScaler` to continuous and ordinal features.
3. **Model Selection:** Train a baseline **Logistic Regression** model using balanced class weights to account for class imbalance.
4. **Goal:** Evaluate predictive accuracy (precision/recall) and examine the coefficients to identify the strongest proxy features associated with a household falling into the SNAP Gap.