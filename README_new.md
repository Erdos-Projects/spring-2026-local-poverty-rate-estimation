# Small Area Estimation of Poverty in the DMV
*A machine learning approach to downscaling aggregate poverty statistics.*

## **Objective**
The American Community Survey (ACS) provides critical socioeconomic data, but at the Census Tract level, estimates often suffer from high Margins of Error (MOEs) due to small sample sizes. This project implements a **Model-Based Post-Stratification** pipeline to "borrow strength" from regional microdata (PUMS) and administrative records (SNAP), producing stable, granular poverty estimates for the Washington-Arlington-Alexandria Metropolitan Area.

## **Modeling information**

1.  **Variable details**
    * **Features:** Age, Sex, Race, and Location (PUMA).
    * **Target:** Binary variable indicating poor or not.
2.  **Train/test split:** We randomly split the data into train and test. 80% of the data is used as training set and remaining 20% as test.

## **Methodology**: Model Definitions
We tested four distinct algorithms to establish our baseline.

| Model | Type | Description | Observation |
| :--- | :--- | :--- | :--- |
| **Logistic Regression** | Linear | **"The Baseline."** A simple statistical model that assumes relationships are straight lines (e.g., probability increases steadily with age). | **0.66 AUC.** Proved that demographics have a strong linear signal, but missed complex interactions. |
| **Random Forest** | Bagging Ensemble | **"The Committee."** Builds hundreds of independent decision trees and averages their votes. Robust against overfitting but hard to calibrate. | **0.67 AUC.** Strong performance, but computationally heavy and produced slightly biased probability estimates. |
| **XGBoost** | Gradient Boosting | **"The Specialist."** Builds trees sequentially, where each new tree tries to fix the errors of the previous one. | **0.65 AUC.** Powerful, but struggled to handle our categorical "Age Bins" without extensive tuning. |
| **LightGBM** | Gradient Boosting | **"The Modern Standard."** A faster, more efficient version of boosting designed specifically for categorical data (like our Age/Race buckets). | **0.68 AUC (Winner).** The best performer. It natively handled the "resolution mismatch" of our binned data and trained 10x faster than the others. |


## **Final Modeling Report**

### The Strategic Trade-off: Ranking vs. Counting
In Small Area Estimation, we face a critical choice between **Discrimination (AUC)** and **Calibration (Bias)**.

* **Model A (High AUC / Biased):** optimized to distinguish poor from non-poor individuals. It achieved an AUC of **0.68** by artificially inflating the risk scores of minority groups. While good for *ranking*, it overestimated the total poverty count by **36%**.
* **Model B (Unbiased / Calibrated):** optimized to represent the true probability of poverty. It achieved a slightly lower AUC of **[INSERT YOUR UNBIASED AUC HERE]**, but reduced the aggregate estimation error to **< 0.1%**.

**Decision:** We selected **Model B (Unbiased LightGBM)**.
* *Reason:* Our goal is to map the *number* of poor people (Estimation), not to identify *specific* individuals (Targeting). The unbiased model provides statistically accurate counts at the PUMA level, making it safe to project onto Census Tracts.


### Conclusion
The final model explains **~66%** of the variance in poverty status using only Age, Sex, and Race. While an "Oracle" model with Job/Education data could achieve **0.78 AUC**, we are limited by the Census Tract variables. The current model is **perfectly calibrated** (Bias ≈ 0) and ready for geographic projection.

* **ETL Pipeline:** See [01_data_pipeline.ipynb](./notebooks/01_data_pipeline.ipynb).
* **Analysis:** See [02_explore_data.ipynb](./notebooks/02_explore_data.ipynb).
* **Modeling:** See [03_model_training.ipynb](./notebooks/03_model_training.ipynb).