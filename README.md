# Food Demand Forecasting and Inventory Planning

## Project Brief

A meal-delivery company operates multiple fulfillment centers and offers different meals across several cities. Because most ingredients are perishable and replenished weekly, inaccurate forecasts can create excess food waste, stockouts, inefficient staffing, and lost sales.

This project develops a machine-learning model to forecast the number of weekly orders for every fulfillment-center and meal combination. The predictions will support procurement, inventory, promotion, and workforce-planning decisions. The results will also be presented through an interactive Looker Studio dashboard for business users.

### Business Question

> How many orders should each fulfillment center expect for every meal during the next 10 weeks?

### Expected Business Value

- Reduce ingredient waste caused by overstocking.
- Reduce lost sales caused by stockouts.
- Improve weekly purchasing and staffing plans.
- Identify meals and centers with volatile or difficult-to-predict demand.
- Evaluate the relationship between pricing, discounts, promotions, and demand.

## Dataset

The project uses the [Food Demand Dataset on Kaggle](https://www.kaggle.com/datasets/arashnic/food-demand).

- Approximately 456,000 training observations
- Historical period: weeks 1–145
- Forecast period: weeks 146–155
- 77 fulfillment centers
- 51 meals
- Prediction target: `num_orders`
- Forecasting level: `week × center_id × meal_id`

### Source Files

| File | Description |
|---|---|
| `train.csv` | Historical weekly demand, prices, and promotions, including `num_orders` |
| `test.csv` | Center-meal combinations for the future 10-week forecast period |
| `meal_info.csv` | Meal category and cuisine information |
| `fulfilment_center_info.csv` | Center type, city, region, and operational area |

### Main Variables

| Variable | Meaning | Modeling Role |
|---|---|---|
| `week` | Week number | Time variable |
| `center_id` | Fulfillment-center identifier | Categorical feature |
| `meal_id` | Meal identifier | Categorical feature |
| `checkout_price` | Final customer price | Numerical feature |
| `base_price` | Standard meal price | Numerical feature |
| `emailer_for_promotion` | Whether the meal received email promotion | Binary feature |
| `homepage_featured` | Whether the meal appeared on the homepage | Binary feature |
| `category` | Meal category | Categorical feature |
| `cuisine` | Cuisine type | Categorical feature |
| `center_type` | Type of fulfillment center | Categorical feature |
| `op_area` | Fulfillment-center operational area | Numerical feature |
| `num_orders` | Number of weekly orders | Prediction target |

## Machine-Learning Approach

This problem is treated as **supervised regression with time-dependent features**. One model will learn across all center-meal combinations, rather than training thousands of separate time-series models.

### Models to Compare

| Model | Role | Reason for Inclusion |
|---|---|---|
| Previous-week demand | Baseline 1 | Tests whether machine learning performs better than simply using last week's demand |
| Four-week moving average | Baseline 2 | Provides a stronger smoothing benchmark |
| Random Forest Regressor | ML benchmark | Captures nonlinear relationships and is easy to explain |
| **LightGBM Regressor** | **Recommended final model** | Efficient on the large dataset and effective with nonlinear, categorical, pricing, promotion, and lag features |
| XGBoost Regressor | Challenger model | Provides a strong gradient-boosting comparison against LightGBM |

### Why LightGBM Is the Primary Model

LightGBM is recommended because the dataset is large, highly categorical, and contains nonlinear interactions between centers, meals, prices, promotions, and historical demand. It is also more practical than training a separate SARIMA or Prophet model for every center-meal combination.

LSTM can be explored as a future extension, but it is not required for the first portfolio version. A well-validated LightGBM model with strong feature engineering will be easier to explain, faster to train, and more appropriate for an entry-level data analytics or data science portfolio.

## Feature Engineering

Planned features include:

- Price difference: `base_price - checkout_price`
- Discount percentage and price ratio
- Demand lags: 1, 2, 4, and 8 weeks
- Rolling demand averages: 4, 8, and 13 weeks
- Rolling demand volatility
- Historical average demand by center and meal
- Promotion interaction features
- Week-based seasonal features
- Meal, cuisine, center, city, and region characteristics

All lag and rolling features must be calculated after shifting the target. This ensures that future demand is never used to predict the past.

## Validation Strategy

The project uses chronological validation instead of a random train-test split:

- Training: weeks 1–135
- Validation: weeks 136–145
- Final forecast: weeks 146–155

This design reproduces the actual 10-week forecasting requirement and prevents time-series leakage.

## Evaluation Metrics

| Metric | Purpose |
|---|---|
| RMSLE | Primary model-comparison metric; reduces the influence of extremely high-demand observations |
| WAPE | Business-friendly measure of total forecast error relative to total demand |
| MAE | Average number of orders missed per prediction |

The final model must outperform both naive baselines before it is considered successful.

## Project Workflow

```mermaid
flowchart LR
    A["Kaggle data"] --> B["Cleaning and EDA"]
    B --> C["Feature engineering"]
    C --> D["Model comparison"]
    D --> E["Looker Studio dashboard"]
```

1. Validate and combine all source tables.
2. Explore demand, centers, meals, prices, and promotions.
3. Create leakage-safe historical and commercial features.
4. Establish naive forecasting baselines.
5. Train and tune Random Forest, LightGBM, and XGBoost.
6. Compare model performance on weeks 136–145.
7. Retrain the winning model and predict weeks 146–155.
8. Analyze feature importance and forecast errors.
9. Export dashboard-ready predictions.
10. Develop operational recommendations in Looker Studio.

## Planned Dashboard

The Looker Studio dashboard will contain:

1. **Executive Overview** — actual demand, predicted demand, forecast accuracy, and weekly trends.
2. **Demand Drivers** — pricing, discount, promotion, meal, cuisine, and center analysis.
3. **Forecast Performance** — WAPE by center and meal, actual versus predicted demand, and major errors.
4. **Inventory Priorities** — high-demand, volatile, and consistently underforecasted meals.

## Repository Structure

```text
food-demand-forecasting/
├── dashboard/
├── data/
│   ├── raw/
│   └── processed/
├── images/
├── models/
├── notebooks/
│   ├── 01_data_cleaning.ipynb
│   ├── 02_exploratory_analysis.ipynb
│   ├── 03_feature_engineering.ipynb
│   └── 04_forecasting_model.ipynb
├── outputs/
├── src/
│   ├── data_preparation.py
│   └── feature_engineering.py
├── .gitignore
├── LICENSE
├── README.md
└── requirements.txt
```

## Setup

Download the four Kaggle files and place them inside `data/raw/`. Then run:

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
pip install -r requirements.txt
python src/data_preparation.py
python src/feature_engineering.py
```

macOS/Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
python src/data_preparation.py
python src/feature_engineering.py
```

## Project Status

- [x] Project scope and repository structure
- [x] Data-preparation starter script
- [x] Leakage-safe feature-engineering starter script
- [ ] Kaggle data downloaded
- [ ] Data cleaning and exploratory analysis
- [ ] Baseline forecasting models
- [ ] LightGBM and XGBoost model comparison
- [ ] Model interpretation and business recommendations
- [ ] Looker Studio dashboard
- [ ] Final portfolio case study

## Author

**Corderitianto Alvian Dwantara**  
Information Systems and Management, BINUS University
