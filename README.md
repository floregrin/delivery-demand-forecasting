# Delivery Demand Forecasting

End-to-end forecasting project for predicting **hourly delivery demand over a 7-day horizon** across geographic hubs and merchant segments, with an operational extension from forecasts to capacity planning.

## Project overview

The analysis uses the public **Delivery Center: Food & Goods Orders in Brazil** dataset and transforms order-level records into 58 hourly Hub × Segment demand series covering January–April 2021.

The project asks four questions:

1. How should hourly demand be constructed from relational marketplace data?
2. Which forecasting approach performs best under leakage-safe rolling-origin validation?
3. How stable are results across forecast windows and training-history choices?
4. How can demand forecasts be translated into operational capacity decisions?

## Demand patterns

Demand is highly structured by both **hour of day** and **merchant segment**. FOOD demand is concentrated around lunch and late-evening peaks, while GOOD demand is substantially lower and flatter.

![Average delivery requests by hour](images/Average%20Delivery%20Requests%20by%20Hour.png)

This intraday structure motivates calendar features and strong seasonal baselines rather than treating observations as exchangeable rows.

## Modeling approach

Three approaches are compared on common 168-hour evaluation windows:

- **Weekly seasonal naive** — a strong `t-168` operational baseline.
- **Local Prophet** — one time-series model per Hub × Segment.
- **Global XGBoost** — one model across all series using location/segment identifiers, calendar features, and only lags known throughout the full 7-day forecast horizon.

A key design choice is avoiding short lags such as `lag_1` and `lag_24` in the direct 168-hour global forecast. Those values would not be observed for most timestamps at prediction time and would introduce leakage unless forecasts were generated recursively.

### Example 7-day holdout

The Rio Hub 8 FOOD series illustrates both the strong weekly/intraday structure and the challenge of reproducing peak magnitude.

![Rio Hub 8 FOOD 168-hour holdout forecast](images/Rio%20Hub%208%20FOOD%20%E2%80%94%20168-hour%20holdout%20forecast.png)

For this illustrative holdout, the seasonal baseline outperformed the local Prophet model. The portfolio-level comparison below therefore evaluates all approaches on identical rolling-origin windows rather than drawing conclusions from one series.

## Headline results

| Model | MAE | wMAPE | Bias (requests/hour) |
|---|---:|---:|---:|
| **Global XGBoost** | **1.4088** | **51.86%** | 0.2304 |
| Seasonal naive | 1.5481 | 56.99% | **0.0928** |
| Local Prophet | 1.9521 | 71.86% | 0.4220 |

Global XGBoost produced the strongest overall point accuracy, reducing weighted absolute error by roughly **9% vs. the seasonal baseline** and **28% vs. local Prophet**. The seasonal baseline retained lower aggregate bias and remained competitive for sparse, low-volume series—an important reminder that model selection should depend on operational context, not only an overall leaderboard.

For FOOD demand, XGBoost achieved 37.3% wMAPE versus 43.0% for seasonal naive and 55.2% for Prophet. Performance was substantially weaker for sparse GOOD series, where percentage metrics are also less stable.

## Backtesting and training-window sensitivity

A single holdout can give a misleading picture of model quality. Rolling backtests therefore evaluate performance over multiple 7-day forecast windows and compare 4-, 6-, and 8-week Prophet training histories on common evaluation periods.

![Training-window sensitivity across forecast windows](images/Training-Window%20Sensitivity%20%E2%80%94%207-Day%20Forecast%20Performance.png)

| Training history | Mean wMAPE | Mean absolute bias |
|---|---:|---:|
| 4 weeks | 38.89% | 1.98 |
| 6 weeks | 37.83% | 1.75 |
| 8 weeks | **37.07%** | **1.48** |

Longer histories improved both average accuracy and bias stability. The chart also shows meaningful variation across forecast windows, reinforcing the need for temporal backtesting rather than a single train/test split.

## Error diagnostics

Forecast error is not constant through time. Comparing MAE with median demand shows that difficult forecast windows are partly associated with changes in demand level and regime.

![Forecast error versus demand level](images/Forecast%20Error%20vs%20Demand%20Level.png)

This is operationally important: an overall average metric can conceal periods in which errors are larger exactly when demand—and therefore staffing exposure—is elevated.

## Operational extension

The analysis also translates FOOD forecasts into a driver-capacity proxy. Historical out-of-sample residuals represent forecast uncertainty, while observed deliveries provide a productivity estimate. A simple expected-cost optimization illustrates the service/capacity trade-off under different shortage-cost assumptions.

Increasing the shortage-cost assumption from 3× to 9× increased average modeled allocation from **4.01 to 5.88 drivers per hub-hour** and estimated demand fulfillment from **85.5% to 91.4%**, at the cost of higher excess capacity.

This is intentionally an operational prototype rather than a production workforce optimizer: the public dataset observes active drivers, not the full scheduled workforce, and does not contain all labor, repositioning, or SLA constraints.

## Repository structure

```text
delivery-demand-forecasting/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md
├── images/
│   └── README figures
└── notebooks/
    ├── 01_delivery_demand_forecasting.ipynb
    └── 02_backtesting_and_window_sensitivity.ipynb
```

## Data

The raw data is **not committed to this repository**. Download the public Kaggle dataset **Delivery Center: Food & Goods Orders in Brazil** and place the required CSV files under:

```text
data/raw/delivery_center/
```

The main analysis uses `orders.csv`, `stores.csv`, `hubs.csv`, `deliveries.csv`, and `drivers.csv` and creates processed artifacts under `data/processed/`.

Dataset: https://www.kaggle.com/datasets/nosbielcs/brazilian-delivery-center

## Running the notebooks

Create an environment and install the dependencies in `requirements.txt`, then start Jupyter from the repository root (or from `notebooks/`; paths are repository-relative).

The notebooks include code using **SoaM / muttlib** for the local Prophet workflow. Availability/install details for those packages may depend on the environment in which the original analysis was developed. The global XGBoost analysis uses standard Python data-science packages.

## What this project demonstrates

- Multi-table data preparation and reconciliation
- Hourly demand aggregation across many related time series
- Leakage-aware feature engineering for multi-step forecasting
- Strong-baseline discipline
- Local statistical forecasting vs. global machine learning
- Rolling-origin validation and segment-level diagnostics
- Training-window sensitivity analysis
- Bias and sparse-series considerations beyond a single accuracy metric
- Translating forecast uncertainty into operational capacity decisions

## Notes

This repository is a portfolio adaptation of a forecasting case study. Company-specific submission language and local-machine paths have been removed; the analysis and model results document the modeling process and conclusions.
