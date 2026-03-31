# Erdős Project: Predicting the SNAP Gap via Socioeconomic Proxies

## 1. Project Objective
Build a predictive model to identify vulnerable households eligible for the Supplemental Nutrition Assistance Program (SNAP) but not receiving it (the "SNAP Gap"). We use observable socioeconomic and demographic proxy features (e.g., household size, education, language barriers, internet access) to classify these households, enabling pure prediction for targeted outreach tools. 

## 2. Team Members
* Aditi Sen
* Zhenyu Yue
* Samia Albalawi
* *Erdos Data Science Bootcamp 2026*

## 3. Data Strategy & Processing
**Data Source:** 2023 American Community Survey (ACS) 1-Year Public Use Microdata Sample (PUMS) for MD, DC, and VA. We use both the Housing (`h`) and Person (`p`) datasets.

**Defining the Target Variable (`TARGET_GAP`):**
We use the entire standard population and construct our target variable using underlying income and SNAP data. 
* **Class 1 (The Gap):** Household is low-income (≤130% poverty line) AND does not receive SNAP.
* **Class 0 (Not in the Gap):** All other households.

**Feature Engineering:** We filter out Group Quarters and households with imputed SNAP values to ensure ground-truth accuracy. We extract extensive proxy features including technological access (internet, smartphones), housing amenities, vulnerability flags (elderly, disabled), and language barriers.

## 4. Exploratory Data Analysis (EDA) Insights
Through our EDA, we discovered that approximately 66% of low-income households in our subset are eligible but not receiving SNAP. This gap indicates barriers beyond income, such as awareness and complexity.

*(Tip: Insert a visual from your PPT here!)*
`![SNAP Gap Distribution](images/snap_gap_chart.png)`

## 5. Modeling & Evaluation
To address the class imbalance (roughly 66% non-gap vs. 34% gap in the initial processing), we implemented the following pipeline:
1. **Data Splitting:** 70% Train, 15% Validate, 15% Test.
2. **Oversampling:** Applied SMOTE (Synthetic Minority Over-sampling Technique) to the training set to balance the classes.
3. **Scaling:** Used `StandardScaler` to normalize continuous/ordinal features to prevent data leakage.
4. **Baseline Model:** Trained a Logistic Regression model with balanced class weights.

**Performance:** * The model achieved a **Validation ROC-AUC Score of 0.77**.
* The Logistic Regression pipeline successfully identifies the majority of "Gap" households, heavily penalizing false negatives.

## 6. Key Takeaways & Actionable Recommendations
Based on our feature correlations and predictive modeling, we recommend the following for policymakers and stakeholders:

* **The "Asset" Association:** The data shows a strong correlation between gap households and protective financial factors (like homeownership and higher education).
* **The Language Factor:** Speaking languages other than English is one of the strongest statistical predictors of falling into the gap, highlighting a clear intersection between language differences and program non-participation.
* **Action Steps:** Prioritize expanding multilingual application support, bilingual caseworkers, and localized community partnerships to engage the most at-risk sub-populations.