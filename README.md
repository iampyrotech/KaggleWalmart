# Walmart Store Sales Forecasting — Kaggle Competition

This repository contains my work for the Kaggle **Walmart Recruiting: Store Sales Forecasting** competition. The goal is to forecast **weekly sales** for each Store–Department combination using historical sales, store metadata, and external features.

In this project, we perform **exploratory data analysis (EDA)**, build a clean merged dataset, engineer seasonal features, and explore **multiple forecasting models**, including regularized regression (glmnet) and Prophet for selected Store/Dept combinations.

Competition link:  
https://www.kaggle.com/competitions/walmart-recruiting-store-sales-forecasting/data

---

## Project Structure

```
/
├── data/                 # train.csv, test.csv, features.csv, stores.csv
├── submissions/          # Kaggle-ready output files
├── notebooks/            # Optional exploration notebooks
├── src/                  # Main modeling and forecasting scripts
└── README.md
```

---

## Workflow

### 1. Data Loading and Merging
We load four datasets:

- `train.csv`  
- `test.csv`  
- `stores.csv`  
- `features.csv`

To avoid duplicate variables like `IsHoliday.x` / `IsHoliday.y`, features are joined after dropping the duplicate `IsHoliday` column. The result is:

- `train_full` — merged, cleaned training data  
- `test_full` — merged, cleaned test data  


### 2. Holiday Engineering
The competition includes “special” holiday dates not captured by the base `IsHoliday` flag.  
We:

1. Build a manual holiday lookup table (Super Bowl, Labor Day, Thanksgiving, Christmas).
2. Join it to the dataset.
3. Create a unified `Holiday` variable with categories:
   - `"SuperBowl"`, `"LaborDay"`, `"Thanksgiving"`, `"Christmas"`, `"None"`


### 3. Seasonal Feature Engineering

To capture yearly patterns, we compute:

- `doy` — day of year  
- `sinDOY`, `cosDOY` — sinusoidal seasonal terms  

These help models capture annual cycles without overfitting.


### 4. Tidymodels Workflow (glmnet)

We prepare a modeling dataset using a `recipe`:

- Mark `Date` as an ID field (not a predictor)  
- Convert `IsHoliday` to numeric  
- Median imputation for numeric predictors  
- Mode imputation for categorical predictors  
- One-hot encoding of categorical variables  
- Remove zero-variance predictors  
- Normalize all numeric predictors  

We tune a **regularized regression model**:

```r
linear_reg(penalty = tune(), mixture = tune()) %>% set_engine("glmnet")
```

Using 5-fold cross-validation, we search over 20 hyperparameter combinations and select the best RMSE.

The finalized workflow produces predictions for the test set and exports:

- `submission.csv`


### 5. Prophet Forecasting for Selected Store–Dept Pairs

We identify the top Store/Department combinations with the most observations and fit separate **Prophet** models:

- Include regressors:  
  `Temperature`, `Fuel_Price`, `CPI`, `Unemployment`, `sinDOY`, `cosDOY`
- Handle missing values with median imputation
- Generate fitted and forecasted values
- Plot training fit + forecast curves for visual inspection

A two-panel figure is produced showing forecasts for the two largest Store/Dept combinations.


---

## Getting Started

1. Place all Walmart data files into `data/`.  
2. Install R dependencies:

```r
install.packages(c(
  "tidyverse", "vroom", "GGally", "patchwork",
  "tidymodels", "recipes", "lubridate", "prophet"
))
```

3. Run the analysis script or Quarto document to reproduce:
   - Merging and cleaning  
   - Feature engineering  
   - glmnet model tuning  
   - Prophet forecasting  
   - Submission file generation  

4. Submission files appear in `submissions/` or project root depending on script settings.


---

## Acknowledgments

- Competition hosted by Kaggle: **Walmart Recruiting — Store Sales Forecasting**  
- Data is originally provided by Walmart  
