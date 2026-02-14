# Small Area Estimation of Poverty in the DMV
*A machine learning approach to downscaling aggregate poverty statistics.*

## Objective
The American Community Survey (ACS) provides critical socioeconomic data, but at the Census Tract level, estimates often suffer from high Margins of Error (MOEs) due to small sample sizes. This project implements a **Model-Based Post-Stratification** pipeline to "borrow strength" from regional microdata (PUMS) and administrative records (SNAP), producing stable, granular poverty estimates for the Washington-Arlington-Alexandria Metropolitan Area.

## Data Sources
This project integrates data from three distinct geographic resolutions:

1.  **Training Data (Individual Level):**
    * **Source:** ACS Public Use Microdata Sample (PUMS).
    * **Content:** Anonymized records (~1% sample) with true labels for Poverty Status (`POVPIP`), Age, Sex, Race, and Location (PUMA).
2.  **Target Data (Tract Level):**
    * **Source:** ACS 5-Year Summary Files.
    * **Content:** Aggregate counts for demographic buckets.
    * **Processing:** We use **Iterative Proportional Fitting (Raking)** to reconstruct the joint demographic distributions (e.g., "Black Female Age 20-24") from the available marginal totals.
3.  **Contextual Features (PUMA Level):**
    * **Source:** USDA SNAP Administrative Records.
    * **Content:** A "Local SNAP Participation Rate" calculated for each PUMA to serve as a high-fidelity economic feature.
4.  **Validation Benchmark (County Level):**
    * **Source:** Census SAIPE (Small Area Income and Poverty Estimates).
    * **Content:** The official gold-standard estimate for county-level poverty, used as a hold-out validation target.

## Methodology: Contextual Post-Stratification
Our approach separates the problem into two distinct stages: **Behavioral Modeling** (estimating risk) and **Population Reconstruction** (estimating composition).

### 1. Population Synthesis (The "Who")
We assume that while the Census *counts* are noisy, the *composition* of a neighborhood is knowable. We use **Raking** to adjust regional PUMS samples until they perfectly match the marginal demographic totals of each specific Census Tract. This creates a synthetic population of "Representative Individuals" for every tract.

### 2. Behavioral Modeling (The "Risk")
We train a classifier to predict the conditional probability of poverty: $P(\text{Poor} | \text{Demographics}, \text{Context})$. We explore several candidate algorithms:
* **Logistic Regression:** A baseline linear model for interpretability.
* **Tree-Based Algorithms (XGBoost/LightGBM):** Capable of capturing non-linear interactions (e.g., "Cliffs" in poverty rates at specific ages).
* **Bayesian Hierarchical Models:** A statistical approach that treats geography as a hierarchy, offering robust shrinkage but higher computational cost.

### 3. Projection (The "Rollup")
We apply the best-performing model to the synthetic population buckets.
* **Step A:** Assign the Tract's specific `Local_SNAP_Rate` to all its synthetic buckets.
* **Step B:** Predict the poverty probability for each bucket.
* **Step C:** Calculate the final estimate:
    $$\text{Tract Poverty} = \sum_{b} (\text{Synthesized\_Count}_b \times \text{Predicted\_Probability}_b)$$

## Validation Framework (KPIs)
Because true poverty counts do not exist at the Census Tract level, we split our validation into two domains:

### A. Internal Validation (ACS Microdata)
*Ground Truth (`is_poor`) **IS** available. We test if the model learns individual behavior correctly.*

| Metric | Definition | Success Threshold |
| :--- | :--- | :--- |
| **AUC-ROC** | **Discrimination.** How well does the model distinguish poor vs. non-poor individuals in the PUMS test set? | `> 0.75` |

### B. External Validation (Tract Estimates)
*Ground Truth is **NOT** available. We validate against proxies, aggregates, and statistical properties.*

| Metric | Definition | Success Threshold |
| :--- | :--- | :--- |
| **SAIPE MSE** | **Calibration.** We aggregate our tract predictions to the County level and compare them against the official Census SAIPE estimates. | `< 0.002` |
| **MOE Hit Rate** | **Consistency.** The percentage of tracts where our smoothed estimate falls within the original ACS Margin of Error. (We want to agree with the Census when it is confident). | `> 60%` |
| **Bootstrap Std Dev** | **Precision.** We resample the training data 50 times to measure the stability of our estimate for each tract. | `< 3%` |
| **Intra-County Variance** | **Granularity.** A heuristic check to ensure the model is finding "pockets of poverty" within counties rather than smoothing everything to the county mean. | `Visible Dist.` |

---
* **ETL Pipeline:** See [01_data_pipeline.ipynb](./notebooks/01_data_pipeline.ipynb).
* **Analysis:** See [02_explore_data.ipynb](./notebooks/02_explore_data.ipynb).
* **Modeling:** See [03_model_training.ipynb](./notebooks/03_model_training.ipynb).