# Delivery Demand Forecasting

End-to-end forecasting project for predicting **hourly delivery demand over a 7-day horizon** across geographic hubs and merchant segments, with an operational extension from forecasts to capacity planning.

## Project overview

The analysis uses the public **Delivery Center: Food & Goods Orders in Brazil** dataset and transforms order-level records into 58 hourly Hub × Segment demand series covering January–April 2021.

The project asks four questions:

1. How should hourly demand be constructed from relational marketplace data?
2. Which forecasting approach performs best under leakage-safe rolling-origin validation?
3. How stable are results across forecast windows and training-history choices?
4. How can demand forecasts be translated into operational capacity decisions?

## Modeling approach

Three approaches are compared on common 168-hour evaluation windows:

- **Weekly seasonal naive** — a strong `t-168` operational baseline.
- **Local Prophet** — one time-series model per Hub × Segment.
- **Global XGBoost** — one model across all series using location/segment identifiers, calendar features, and only lags known throughout the full 7-day forecast horizon.

A key design choice is avoiding short lags such as `lag_1` and `lag_24` in the direct 168-hour global forecast. Those values would not be observed for most timestamps at prediction time and would introduce leakage unless forecasts were generated recursively.

## Headline results

| Model | MAE | wMAPE | Bias (requests/hour) |
|---|---:|---:|---:|
| **Global XGBoost** | **1.4088** | **51.86%** | 0.2304 |
| Seasonal naive | 1.5481 | 56.99% | **0.0928** |
| Local Prophet | 1.9521 | 71.86% | 0.4220 |

Global XGBoost produced the strongest overall point accuracy, reducing weighted absolute error by roughly **9% vs. the seasonal baseline** and **28% vs. local Prophet**. The seasonal baseline retained lower aggregate bias and remained competitive for sparse, low-volume series—an important reminder that model selection should depend on operational context, not only an overall leaderboard.

For FOOD demand, XGBoost achieved 37.3% wMAPE versus 43.0% for seasonal naive and 55.2% for Prophet. Performance was substantially weaker for sparse GOOD series, where percentage metrics are also less stable.

## Backtesting and training-window sensitivity

Rolling backtests evaluate temporal stability and compare 4-, 6-, and 8-week Prophet training histories on common forecast periods.

| Training history | Mean wMAPE | Mean absolute bias |
|---|---:|---:|
| 4 weeks | 38.89% | 1.98 |
| 6 weeks | 37.83% | 1.75 |
| 8 weeks | **37.07%** | **1.48** |

Longer histories improved both accuracy and bias stability.

## Operational extension

The analysis also translates FOOD forecasts into a driver-capacity proxy. Historical out-of-sample residuals represent forecast uncertainty, while observed deliveries provide a productivity estimate. A simple expected-cost optimization illustrates the service/capacity trade-off under different shortage-cost assumptions.

This is intentionally an operational prototype rather than a production workforce optimizer: the public dataset observes active drivers, not the full scheduled workforce, and does not contain all labor, repositioning, or SLA constraints.

## Repository structure

```text
delivery-demand-forecasting/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md
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
