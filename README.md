# M5 Retail Demand Forecasting
### XGBoost-based SKU-level sales forecasting for retail planning

---

## Business Context

Accurate demand forecasting is the foundation of retail planning. It drives:
- **Open-to-Buy (OTB) budgets** — how much to purchase per category per season
- **Replenishment decisions** — when and how much stock to move to each store
- **Inventory redistribution** — rebalancing stock across locations based on forward demand
- **Supplier PO planning** — aggregating store-level demand to DC and supplier order level

This project builds a production-style demand forecasting pipeline using the M5 Forecasting Competition dataset — 3,049 SKUs across Walmart's CA_1 store, with 5+ years of daily sales history.

The methodology is grounded in real-world retail experience — having built and deployed demand planning engines across 900+ stores in large-scale retail environments in the GCC.

---

## Dataset

**M5 Forecasting Competition** (Kaggle)
- 30,490 item-store combinations across 3 US states and 10 Walmart stores
- 1,913 days of daily unit sales history (2011–2016)
- Supplementary files: calendar (events, holidays), sell prices (weekly)
- This notebook scopes to CA_1 store (3,049 SKUs) for computational efficiency
- Same pipeline applies identically across all 10 stores

---

## Approach

### 1. Data Pipeline
- Load and filter raw M5 files
- Reshape from wide format (1 row per item) to long format (1 row per item per day)
- Merge calendar (real dates, events, holidays) and sell prices

### 2. Outlier Treatment
- Winsorize sales at 99th percentile per item
- Removes promotional spikes and impulse buys that distort model training
- Result: 39,631 outliers capped (0.68% of data), RMSE improved 6.1%

### 3. Feature Engineering
| Feature | Business Rationale |
|---|---|
| lag_7, lag_28, lag_365 | Demand memory — recent and seasonal history |
| roll_mean_7, roll_mean_28 | Smoothed trend signal |
| day_of_week, is_weekend | Weekly sales patterns |
| month, week_of_year | Seasonality |
| sell_price, price_vs_avg | Price elasticity and promotional signals |
| has_event | Holiday and event-driven demand |

### 4. Model — XGBoost Regressor
- Trained on 4.6M observations (2012–2016)
- Validated on final 28 days across all 3,049 items
- 28-day forecast horizon (standard retail OTB planning cycle)

---

## Results

| Metric | Value |
|---|---|
| Validation RMSE (before outlier treatment) | 2.04 units/day |
| Validation RMSE (after outlier treatment) | 1.91 units/day |
| Improvement from data cleaning | 6.1% |
| Forecast horizon | 28 days |
| Items forecast | 3,049 SKUs |
| Top predictive feature | 7-day rolling mean sales |

---

## Key Findings

- **Recent sales history dominates** — 7-day and 28-day rolling means are the strongest demand signals, confirming that demand has strong momentum in retail
- **Outlier treatment matters** — 6.1% RMSE improvement from winsorization alone, before any model tuning
- **Weekend effect is significant** — is_weekend ranks as 3rd most important feature
- **Price signal is present but moderate** — price elasticity exists but is category-dependent

---

## Limitations and Next Steps

- Forecast smoothing on low-velocity items (<2 units/day) — expected in SKU-level retail forecasting
- Promotional spike detection requires event interaction features (lag x has_event)
- Production model would extend to all 10 stores using identical pipeline
- LSTM layer on top would improve spike detection for promotional demand
- WRMSSE scoring (official M5 metric) to be implemented for benchmarking

---

## Repository Structure

- `m5_demand_forecasting.ipynb` — Main notebook
- `forecast_output_ca1_28day.csv` — 28-day forecast output  
- `forecast_results.png` — Model evaluation charts
- `xgboost_m5_forecast_model.pkl` — Saved trained model
- `Data/` — Raw M5 dataset files

---

## Tech Stack

Python, XGBoost, Pandas, NumPy, Scikit-learn, Matplotlib, Seaborn

---

## Author

**Sagar Sharma**  
AI & Analytics Consultant | Retail Demand Forecasting  
[Kaggle](https://www.kaggle.com/sagarsharmafeb87) | [LinkedIn](https://www.linkedin.com/in/sagar-sharma-a6211589/)
