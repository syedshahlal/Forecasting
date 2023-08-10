## Problem Statement

Your client operates a meal delivery company with multiple fulfillment centers in various cities. They require assistance in forecasting demand for the next 10 weeks (Weeks: 146-155) for center-meal combinations in the test set. You have historical demand data, product features (category, price, discount, etc.), and fulfillment center information.

### Data Dictionary

1. **Weekly Demand Data (train.csv)**: Historical demand data for all centers (Weeks: 1 to 145)

| Variable | Definition |
| -------- | ---------- |
| id | Unique Identifier |
| week | Week Number |
| center_id | Fulfillment Center ID |
| meal_id | Meal ID |
| checkout_price | Final price (including discount, taxes, and delivery charges) | 
| base_price | Base price of the meal |
| emailer_for_promotion | Emailer sent for meal promotion |
| homepage_featured | Meal featured on homepage |
| num_orders | (Target) Number of Orders |

2. **Fulfillment Center Info.csv**: Information about each fulfillment center

| Variable | Definition |
| -------- | ---------- |
| center_id | Fulfillment Center ID |
| city_code | Unique code for the city | 
| region_code | Unique code for the region |
| center_type | Anonymized center type |
| op_area | Operational area (in square kilometers) |

3. **Meal Info.csv**: Information about each meal being served

| Variable | Definition |
| -------- | ---------- |
| meal_id | Unique ID for the meal |
| category | Meal type (beverages/snacks/soups, etc.) |
| cuisine | Meal cuisine (Indian/Italian, etc.) |

### Evaluation Metric

The evaluation metric for this competition is 100 times the RMSLE (Root of Mean Squared Logarithmic Error) across all entries in the test set. Test data is divided into Public (30%) and Private (70%) data.

## Solution

To address this time series problem, the following steps were taken:

#### Data Transformation:

1. Apply a log transformation to the target variable (number of orders) due to its right-skewed distribution.
2. Also, apply log transformation to base_price, checkout_price, and num_orders.

#### Feature Engineering:

1. Calculate the difference between base_price and checkout_price for each record.
2. Compute the difference between the previous week's checkout_price and the current week's checkout_price.
3. Create lag features with lags of 10, 11, and 12 weeks.
4. Compute exponentially weighted means over the last 10, 11, and 12 weeks.

#### Cross-Validation Strategy:

- Utilize the last 10 weeks (Weeks 136-145) of each center-meal pair data as a validation dataset from the train dataset.

#### Model:

1. Employ a single CatBoost model with a high level of regularization to prevent overfitting due to new features derived from the target variable.
2. Achieve an RMSLE of 0.54 with this model.

#### What Didn't Work:

1. Using only the original data with a boosting regressor resulted in an RMSLE of 1.58.
2. Using only the difference between base_price and checkout_price as features, without any lag or exponentially weighted features, yielded poor results.
3. Using rolling mean and median features over the last 26, 52, and 104 weeks had low feature importance and did not contribute significantly.
4. Geographical features had low feature importance, so they were not included in the final model.

#### TODO / Improvements:

1. Conduct extensive hyperparameter tuning and feature selection.
2. Generate more features based on categorical encoding methods like mean encoding, frequency encoding, and hash encoding.
3. Explore additional algorithms such as XGBoost, LightGBM, Linear Regression, etc.
4. Consider time series models like ARIMA and Prophet.
5. Experiment with ensemble techniques using different models for improved performance.
