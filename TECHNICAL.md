# Notes

Design choices that did not fit cleanly in the notebook.

**Programme:** Time Series Analysis and Forecasting (SDAIA Academy)  
**Cohort dates:** 13–15 September 2026

## Pipeline

1. Download the UCI Metro Interstate Traffic Volume CSV (gzip) over HTTPS. No API key.
2. Parse `date_time`, drop duplicate hours (the raw file repeats a timestamp when several weather labels were logged).
3. Treat physically impossible readings as missing: temperature below 200 K (the known 0 K error) and `rain_1h` above 100 mm (the known 9 831 mm spike).
4. Keep a contiguous hourly window from 2018-01-01 through 2018-09-30. The 2012–2015 record has a long gap; a regular frequency is required for STL, ACF, and Holt-Winters.
5. Reindex to a complete hourly calendar and interpolate numeric columns for gaps of at most six hours. Longer holes, if any, are dropped from the modelling frame.
6. Target: `traffic_volume`. Forecast horizon: 24 hours. Daily seasonal period: 24. Weekly structure enters LightGBM as `lag_168` and a 168-hour rolling mean.

## Leakage rules

- Lag and rolling features are built from `y.shift(1)` (or longer lags). The current hour’s volume never appears as a feature for that hour’s prediction.
- Weather (`temp`, `rain_1h`, `snow_1h`, `clouds_all`) is lagged by one hour. A 24-hour forecast does not assume we already know future weather.
- Calendar features (hour, weekday, month, cyclic encodings) and the holiday flag are treated as known future information — the same assumption Prophet makes for holidays.
- Recursive multi-step LightGBM forecasts feed *predicted* volume back into lags for steps 2…24. They do not peek at actual test-window volume.
- `casual`-style leakage does not apply here; there is a single target.

## Backtest design

Splits come from `expanding_window_splits` in the course’s `common/backtest.py`. I used an expanding window: I-94’s daily/weekly shape is stable across 2018, so extra history should help.

- Horizon: 24 hours (one diurnal cycle; the operational question is “what does tomorrow look like hour by hour”).
- Folds: 4.
- Minimum train size: 90 days (2 160 hours), enough for `lag_168` plus a 168-hour rolling window.
- Seasonal-naive and Holt-Winters are scored with `run_backtest`. LightGBM uses the same slices on the feature table so calendar/holiday columns can be attached without changing fold boundaries.
- No fold’s training slice overlaps its own test window.

## Metrics

MAE and RMSE are reported in vehicles per hour. The scale-free companion is **MASE** with `seasonal_period=24`, scaled against the in-sample seasonal-naive MAE on each fold’s training history (`common.metrics.mase`).

MAPE is not used as the headline metric. Volume is continuous and almost always far from zero, so MAPE would not blow up — but it is still a poor comparison to the seasonal-naive baseline that MASE is defined against. WAPE is available in `metrics.py` and would be the right scale-free choice on an intermittent series; this series is not intermittent.

## Interval calibration

LightGBM has no native interval. An 80% conformal (split-conformal / residual-quantile) interval is built on each fold:

1. The training window is split into a fit piece and a trailing calibration piece (the last 7 days of train).
2. Absolute residuals on the calibration piece set the half-width at the 80% quantile (the 90th percentile of |error|, which yields a symmetric 80% interval).
3. That width is added to / subtracted from the 24-hour point forecast.
4. `coverage()` and `interval_width()` from `common/metrics.py` are reported against the nominal 80% level. Coverage alone is not treated as success.

## Model families compared

**What the backtest showed on this series.** Seasonal-naive (repeat yesterday’s hourly profile) is the floor. Holt-Winters with a *daily* seasonal period cannot separate weekdays from weekends; LightGBM’s `lag_168` and weekday features can. The executed notebook’s fold table is the evidence: LightGBM mean MASE 0.44 vs seasonal-naive 0.90 vs Holt-Winters 3.01. That is why the written recommendation deploys LightGBM with a conformal interval, and keeps Holt-Winters only as the explanation of the diurnal shape.

| Family | Role |
| --- | --- |
| Seasonal-naive (period 24) | Baseline every model is expected to beat (`seasonal_naive_forecast`) |
| Holt-Winters / ETS (additive trend + additive daily seasonality) | Required classical model; residual Ljung-Box on a single in-sample fit |
| LightGBM | Required GBM; lag / rolling / calendar / lagged-weather features |

The notebook recommendation uses those numbers plus whether the forecast is explainable and whether the interval is usable. Accuracy is one input, not the whole decision.

## Runtime fetch

The notebook does not vendor `common/metrics.py` or `common/backtest.py`. The setup cell copies the lab `fetch()` helper: local checkout first, otherwise `raw.githubusercontent.com/MohammadYusif/time-series-forecasting-ai-systems/main/...`. Those files are listed in `.gitignore` so a Colab download is not committed back.
