# Model Evaluation and Hyperparameter Methodology

## 1. Algorithm Overview & Comparison

### Logistic Regression (Baseline)
* **Features:** A linear statistical model that applies a sigmoid function to predict the probability of a binary outcome.
* **Pros:** Highly interpretable; coefficients directly explain the direction and magnitude of each proxy's impact. Fast to train.
* **Cons:** Assumes linear relationships. Cannot organically capture complex intersections of variables (e.g., the combined penalty of having no vehicle and a large household).

### Random Forest (Bagging Ensemble)
* **Features:** An ensemble method that trains hundreds of deep, independent decision trees in parallel on random subsets of data and features, then averages their predictions.
* **Pros:** Highly robust to overfitting. Naturally captures non-linear relationships and complex variable interactions. 
* **Cons:** Operates as a "black box" requiring secondary explainers (like SHAP) for directional interpretability. Can struggle with severe minority classes unless explicitly weighted.

### XGBoost (Level-Wise Gradient Boosting)
* **Features:** A sequential ensemble model where each new, shallow tree is built specifically to correct the residual errors of the previous trees. Grows trees horizontally (level-wise).
* **Pros:** Consistently delivers top-tier predictive accuracy. Includes built-in L1/L2 regularization to penalize overly complex models.
* **Cons:** Computationally intensive. Highly sensitive to hyperparameters; prone to overfitting on imbalanced data if not strictly regulated.

### LightGBM (Leaf-Wise Gradient Boosting)
* **Features:** Similar to XGBoost, but grows trees vertically (leaf-wise), choosing to split the leaf that will yield the maximum decrease in loss, regardless of tree depth.
* **Pros:** Exceptionally fast training speeds and highly memory-efficient. Optimized for large datasets and imbalanced classes.
* **Cons:** The leaf-wise growth algorithm makes it highly susceptible to overfitting on smaller datasets if tree complexity is not strictly capped.

---

## 2. Hyperparameter Selection & Ranges

The hyperparameter search spaces were constrained based on the dataset's specific architecture: moderate row count (~42,000), low feature space (~15 proxies), and severe class imbalance (~8% minority class).

**Global Parameters (All Tree Models)**
* **`n_estimators`:** The total number of individual decision trees built within the ensemble.
  * *Range:* `[100, 500]`
  * *Depends on:* Dataset size and learning rate. Going beyond 500 trees on a dataset of this size rarely improves generalization and strictly increases computational cost.

**Random Forest Parameters**
* **`max_depth`:** The maximum number of levels (splits) a single tree can grow from its root to its final leaves.
  * *Range:* `[5, None]`
  * *Depends on:* Regularization needs. `None` allows pure leaves, while lower numbers force the model to identify broader demographic patterns.
* **`max_features`:** The maximum number of proxy variables a tree is allowed to evaluate when deciding where to make a split.
  * *Range:* `['sqrt', 'log2']`
  * *Depends on:* Total feature count ($p$). Restricts how many proxies a tree evaluates per split, forcing diversity among the trees.
* **`min_samples_split`:** The minimum number of households required in a node before it is allowed to split again.
  * *Range:* `[2, 10]`
  * *Depends on:* Dataset noise (prevents splitting on outliers).
* **`min_samples_leaf`:** The minimum number of households that must end up in a final, terminal leaf after a split.
  * *Range:* `[1, 4]`
  * *Depends on:* Dataset noise. Prevents the model from creating splitting rules that apply to highly isolated outlier households.

**Boosting Parameters (XGBoost & LightGBM)**
* **`learning_rate`:** The multiplier that scales down the contribution of each new tree, forcing the model to learn gradually.
  * *Range:* `[0.01, 0.1]`
  * *Depends on:* `n_estimators`. Lower learning rates require more trees but yield a more generalizable model via the shrinkage principle.
* **`max_depth`:** The maximum number of levels (splits) a single tree can grow from its root to its final leaves.
  * *Range:* `[3, 7]`
  * *Depends on:* Overfit risk. Boosting algorithms require "weak learners." Deep trees in sequential boosting lead to catastrophic overfitting.
* **`num_leaves` (LightGBM):** The absolute maximum number of terminal leaves allowed in a single LightGBM tree.
  * *Range:* `[15, 50]`
  * *Depends on:* `max_depth`. Must be mathematically constrained below $2^{max\_depth}$ to prevent the leaf-wise algorithm from memorizing data.
* **`min_child_weight` (XGBoost):** The minimum required sum of data point "weights" inside a node to justify making a new split.
  * *Range:* `[1, 5]`
  * *Depends on:* Class imbalance. Demands a minimum sum of instance weight in a child node to halt unhelpful, hyper-specific splits.
* **`subsample`:** The percentage of the total dataset randomly chosen to train each individual tree.
  * *Range:* `[0.7, 1.0]`
  * *Depends on:* The need for injected randomness (bagging) to prevent overfitting.
* **`colsample_bytree`:** The percentage of the total proxy features randomly chosen to build each individual tree.
  * *Range:* `[0.7, 1.0]`
  * *Depends on:* The need to ensure the model evaluates secondary demographic signals rather than relying entirely on the single strongest feature.

---

## 3. Academic & Industry References

The methodologies and hyperparameter constraints applied in this analysis are based on established statistical learning heuristics:

1. **Géron, A. (2022).** *Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow* (3rd ed.). O'Reilly Media.
   * *Application:* Justification of standard grid-search ranges (e.g., estimator limits and baseline depths) for applied ensemble modeling.
2. **Hastie, T., Tibshirani, R., & Friedman, J. (2009).** *The Elements of Statistical Learning: Data Mining, Inference, and Prediction* (2nd ed.). Springer.
   * *Application:* Mathematical justification for Random Forest feature subsampling. Page 588 defines the $\lfloor\sqrt{p}\rfloor$ rule for classification tasks.
3. **Friedman, J. H. (2001).** *Greedy Function Approximation: A Gradient Boosting Machine.* Annals of Statistics.
   * *Application:* Theoretical basis for the inverse relationship between learning rate (shrinkage) and estimator count in gradient boosting.
4. **Microsoft Corporation.** *LightGBM Official Documentation: Parameters Tuning Guide.*
   * *Application:* Parameter bounding rules; specifically the dictate that `num_leaves` must remain strictly less than $2^{max\_depth}$ to control algorithm complexity.