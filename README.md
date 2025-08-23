# Time Series Forecasting — Mini Project (ETS, ARIMA, DHR, LSTM)

**TL;DR.** On `AAPL` daily prices (2019-01-01–2023-12-31), with a single 80/20 hold-out (H=251), **LSTM (lookback=128)** achieved **RMSE 4.16 (↓87.4% vs seasonal-naive m=252)**. Classical **ETS/ARIMA/DHR** underperformed the seasonal-naive baseline on this split.

---

## Results

*Data:* Yahoo Finance (Adjusted Close) • *Freq:* trading days • *Season length:* `m = 252`  
*Metrics:* lower is better (sMAPE, RMSE, MAE, MASE) • `Δ vs Seasonal-Naive (RMSE)` uses ↑ (worse) / ↓ (better)

| Model                         | sMAPE |  RMSE |   MAE |  MASE | Δ vs Seasonal-Naive (RMSE) |
|------------------------------|-----:|-----:|-----:|-----:|:---------------------------:|
| Seasonal-Naive (m=252)       | 17.65 | 32.94 | 28.48 | 0.780 | — |
| ETS (mul)            | 24.26 | 40.46 | 37.33 | 1.023 | ↑22.8% |
| ARIMA (2,1,2)\_252           | 28.47 | 46.63 | 43.30 | 1.186 | ↑41.6% |
| DHR (K=3) + ARMA(1,1) errors | 25.03 | 40.97 | 38.43 | 1.053 | ↑24.4% |
| **LSTM (lookback=128)**      | **2.06** | **4.16** | **3.46** | **0.095** | **↓87.4%** |

<img width="2400" height="1200" alt="image" src="https://github.com/user-attachments/assets/a1d45b85-e421-4feb-9c76-0e036b59c223" />


**Sanity checks for the LSTM gain**
- Scaler fit on **train only**; no leakage from val/test.  
- Val/test sequences include the lookback overlap (`start = split_index - lookback`).  
- Forecasts aligned to `test.index`; metrics computed on **original scale**.  
- Re-checked with **rolling-origin CV** (recommend adding a CV table/plot).



---

## Data

**Source.** Yahoo Finance via `yfinance` (Adjusted Close → `y`, Date → `ds`).  
**Range.** 2019-01-01 to 2023-12-31 (downloaded with `end="2024-01-01"`).  
**Why Adjusted Close?** Handles splits/dividends for continuity.

---

## Evaluation Protocol
- **Split:** 80% train / 20% test (H=251).
- **Validation:** last ~10% of the **training window** held out for early stopping (no leakage).
- **Seasonality:** trading-year `m=252`.
- **Baselines:** Seasonal-Naive (m=252).
- **Metrics:** RMSE, MAE, sMAPE, MASE (scaled by in-sample seasonal-naive error).

---

## Models

- **ETS (mul, no damping):** Holt–Winters with **multiplicative trend**, `seasonal='mul'` when all y>0 (else `'add'`); **no** `damped_trend`.  
- **ARIMA (2,1,2):** non-seasonal ARIMA on the **Box–Cox** transformed series; inverse Box–Cox for evaluation.  
- **DHR:** Fourier seasonality (`period=252`, `K=3`) + ARMA(1,1) errors.  
- **LSTM:** univariate sequence model, lookback=128, scaler fit on train only, validation early stopping.

<details>
<summary>Practical Notes (stationarity, diagnostics, etc.)</summary>

- **Stationarity & transforms:** Box-Cox/log → differencing (as needed) → ADF+KPSS.  
- **Diagnostics:** ACF/PACF, residual diagnostics; Ljung–Box; for GARCH (volatility) use squared-residual checks.  
- **GARCH:** modeled separately on **returns** (not included in the mean-forecast table).  
- **Tips:** Start with baselines; keep Fourier `K` modest (2–4) to avoid overfit; avoid SARIMA with very large `m` (use DHR instead).
</details>

---
## Core concepts I practiced

- **Stationarity & transforms:** Box–Cox / log → (seasonal) differencing → ADF checks.
- **ETS / Holt–Winters:** level–trend–seasonality (additive/multiplicative), with/without damping.
- **ARIMA:** ARIMA for non-seasonal dynamics.
- **Dynamic Harmonic Regression (DHR/TBATS):** Fourier seasonality (e.g., period=252, K=3) + ARIMA errors; supports multiple/non-integer cycles.
- **GARCH (on returns):** volatility/uncertainty modeling (risk bands), not mean level forecasts.
- **LSTM:** sliding windows (lookback=128), train-only scaling, held-out validation for early stopping, leakage control.
- **Diagnostics:** residual ACF/PACF, Ljung–Box, QQ plots; for GARCH, squared-residual autocorrelation.
- **Evaluation:** blocked rolling-origin backtests; metrics **RMSE/MAE/sMAPE/MASE**; compare to Naive & Seasonal-Naive.


---

## Reproducibility

```bash
# Environment
conda create -n venv python=3.11.13 -y
conda activate venv
pip install -r requirements.txt
