# Notes

Design choices that did not fit cleanly in the notebook.

**Programme:** Time Series Analysis and Forecasting (SDAIA Academy)  
**Cohort dates:** 13–15 September 2026

## Pipeline

1. Download the UCI Metro Interstate Traffic Volume CSV (gzip) over HTTPS. No API key.
2. Parse `date_time`, drop duplicate hours (the raw file repeats a timestamp when several weather labels were logged).
3. Treat physically impossible readings as missing: temperature below 200 K (the known 0 K error) and `rain_1h` above 100 mm (the known 9 831 mm spike).
4. Keep a contiguous hourly window from 2018-01-01 through 2018-09-30. The 2012–2015 record has a long gap; a regular frequency is required for STL, ACF, and Holt-Winters.
5. Reindex to a complete hourly calendar and interpolate numeric columns for gaps of at most six hours.
6. Target: `traffic_volume`. Horizon: 24 hours. Daily seasonal period: 24. Weekly structure enters LightGBM as `lag_168`.

## Leakage rules

- Lag and rolling features use `y.shift(1)` (or longer lags). The current hour’s volume is never a feature for that hour.
- Weather is lagged by one hour.
- Calendar features and holidays are treated as known future information.
- Recursive LightGBM feeds predictions back into lags for steps 2…24. It does not read test-window volume.

## Backtest design

Both `expanding_window_splits` and `rolling_window_splits` from `common/backtest.py`, 4 folds, 24-hour horizon. Expanding grows the train set; rolling keeps a 90-day window. Seasonal-naive, Holt-Winters, and SARIMAX go through `run_backtest`. LightGBM uses the same slices on the feature table.

SARIMAX is fit on the last 28 days of each training window. Hourly seasonal MLE on 6,000+ points is too slow to refit from scratch on every fold.

## Metrics

MAE and RMSE in vehicles per hour, plus MASE with `seasonal_period=24`. MAPE is not the headline metric — volume is almost never zero, but MASE is the comparison to seasonal-naive.

## Intervals

Three 80% intervals, scored with `coverage()` and `interval_width()`:

- Conformal around LightGBM (last 7 training days as calibration).
- Prophet `yhat_lower` / `yhat_upper` on the last expanding fold.
- sktime `NaiveForecaster(sp=24)` `predict_interval` / `predict_quantiles`, with pinball loss on the 0.1 / 0.5 / 0.9 quantiles.

## Model families

- **statsmodels (Holt-Winters)** — readable daily seasonal. Fails on weekends.
- **statsmodels (SARIMAX)** — ARIMA-family fit. `d=0` from ADF. Order from a small AIC grid on 21 days of hourly data.
- **LightGBM** — what I would deploy here, with the conformal interval.
- **Prophet** — native interval, built for daily data with holidays. I would use it on a daily series, not this hourly counter.
- **sktime** — the interval/quantile API. The naive wrapper is a check, not a production model.

Expanding vs rolling is decided in the notebook from the MASE table, not in advance.

## Runtime fetch

The setup cell loads `metrics.py` and `backtest.py` from a local checkout if present, otherwise from `raw.githubusercontent.com/MohammadYusif/time-series-forecasting-ai-systems`. Those downloads are gitignored.
