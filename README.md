# Australian Unemployment Forecasting

> Macroeconomic time series analysis forecasting Australian unemployment using STR decomposition, ARIMA, Gradient Boosting, and SARIMAX with interest rate scenario modelling.

![Python](https://img.shields.io/badge/Python-3.11-yellow)
![statsmodels](https://img.shields.io/badge/statsmodels-time%20series-blue)
![sklearn](https://img.shields.io/badge/scikit--learn-ML-orange)
![Pandas](https://img.shields.io/badge/Pandas-data%20wrangling-150458)

---

## Overview

This project analyses the relationship between Australia's key macroeconomic indicators — unemployment rate, CPI, and the RBA cash rate — and builds forecasting models to predict unemployment trends under different monetary policy scenarios.

Data spans 2011–2024 from official Australian government sources (ABS and RBA), covering the post-GFC recovery, COVID-19 shock, and post-pandemic tightening cycle.

---

## Dataset

| Series | Source | Frequency | Range |
|---|---|---|---|
| Unemployment Rate | ABS Labour Force Survey | Monthly | 1978–2025 |
| CPI | ABS Consumer Price Index | Quarterly | 1948–2025 |
| RBA Cash Rate | Reserve Bank of Australia | Daily | 2011–2025 |

**Analysis window:** March 2011 – December 2024 (166 observations after alignment)

**Preprocessing:** RBA daily cash rate resampled to monthly (end-of-month), CPI forward-filled from quarterly to monthly, all series aligned to common end-of-month timestamps.

---

## Analysis Pipeline

```
Raw Data (ABS + RBA)
       │
       ▼
┌─────────────────┐
│  Data Wrangling │  Resample, align, join 3 series to monthly frequency
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  EDA & Corr.    │  Correlation matrix, distributions, rolling statistics
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ STR Decomp.     │  Manual trend (spline) + seasonal + remainder extraction
└────────┬────────┘  Validated against statsmodels STL
         │
         ▼
┌─────────────────┐
│ ARIMA Modelling │  AIC grid search → ARIMA(2,2,1) on trend component
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ GBT Forecasting │  Gradient Boosting with bootstrap uncertainty estimation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ SARIMAX         │  Cash rate as exogenous variable
└────────┬────────┘  2 interest rate scenarios (cut vs hold)
         │
         ▼
  Forecast Comparison & Policy Insights
```

---

## Results

### Correlation Analysis

| Pair | Correlation | Interpretation |
|---|---|---|
| Unemployment vs CPI | -0.649 | Supports Phillips Curve — lower unemployment accompanies higher inflation |
| Unemployment vs Cash Rate | -0.394 | Counter-cyclical policy — RBA cuts rates during rising unemployment |
| CPI vs Cash Rate | +0.045 | Weak direct relationship |

---

### STR Decomposition

Manual decomposition implemented from scratch using spline smoothing for trend and month-based seasonal averaging. Validated against statsmodels STL.

- **Reconstruction error (MAE): 0.000000** — perfect reconstruction confirmed
- Remainder is stationary (ADF p-value: 0.0117)
- Seasonal peaks in Jan–Feb (+0.46% to +0.70%), troughs in Nov–Dec (-0.50% to -0.58%)

---

### Model Comparison (36-month forecast: 2022–2024)

| Model | MAE | RMSE | MAPE |
|---|---|---|---|
| **ARIMA(2,2,1) + STR** | **1.1823 pp** | **1.2976 pp** | **30.84%** |
| Gradient Boosting + STR | 1.8102 pp | 1.8348 pp | 46.92% |

**ARIMA outperforms Gradient Boosting by 29.3% (RMSE).** The high MAPE reflects the difficulty of forecasting the sharp post-COVID unemployment drop from ~7% to ~3.5% — a structural break not captured in pre-2022 training data.

---

### SARIMAX Scenario Analysis (Jan–Jun 2025)

Two RBA monetary policy scenarios modelled using SARIMAX with cash rate as exogenous variable:

| Scenario | Cash Rate | Mean Forecast | Jun 2025 |
|---|---|---|---|
| Rate Cut (-0.75pp) | 3.60% | 3.71% | 3.68% |
| Hold | 4.35% | 3.58% | 3.55% |
| Difference | — | +0.13pp | +0.13pp |

**Finding:** The rate cut scenario paradoxically forecasts marginally higher unemployment (+0.13pp), consistent with the cash rate coefficient being statistically insignificant (p=0.99) — suggesting lagged or indirect effects not captured in this model window.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| pandas / numpy | Data wrangling & alignment |
| statsmodels | STL decomposition, ARIMA, SARIMAX, ADF test |
| scikit-learn | Gradient Boosting, MinMaxScaler, metrics |
| scipy | Spline smoothing for trend estimation |
| matplotlib / seaborn | Visualisation |


## Project Structure

```
australia-unemployment-forecasting/
├── unemployment_forecasting.ipynb  ← Full analysis (all tasks)
├── dataset/                           ← ABS + RBA source files (not committed)
└── README.md
```

---

## Key Findings

- Australia's unemployment followed a clear cycle: gradual rise 2011–2015 → stable 2016–2019 → COVID spike 2020 → record low ~3.5% by 2023 → mild rebound 2024
- **ARIMA(2,2,1)** selected via AIC grid search outperforms Gradient Boosting for this series
- Strong lag-1 dependence in GBT (importance: 0.92) confirms unemployment is highly autoregressive
- Cash rate has a statistically insignificant direct effect on unemployment in this model — consistent with known policy transmission lags of 12–18 months
- Seasonal pattern shows unemployment peaks in Q1 (Jan–Feb) and troughs in Q4 (Nov–Dec)
