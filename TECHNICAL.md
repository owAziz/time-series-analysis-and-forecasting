# Notes

SDAIA Academy — Time Series Analysis and Forecasting  
13–15 September 2026

I kept 2018 hourly I-94 after dropping duplicate timestamps, a 0 K temperature, a bogus rain spike, and the 2014–2015 hole. Tiny gaps (≤ 6 hours) are interpolated. Target is `traffic_volume`, 24 hours ahead.

Lags and rolling stats only use past volume. Weather is lagged one hour. Calendar and holidays are known in advance. When LightGBM walks 24 steps, it feeds its own predictions back in — not the holdout.

I walked four 24-hour holdouts two ways: a training set that grows, and a 90-day window that slides. SARIMAX is refit on the last 28 days of each train window because a full-length hourly seasonal fit is slow.

Errors are MAE, RMSE, and MASE against yesterday’s hourly profile.

Intervals: residual quantiles around LightGBM, Prophet’s band on the last holdout, and sktime’s seasonal-naive band on the same holdout.

I would ship LightGBM on the growing window. Holt-Winters and SARIMAX are the statsmodels checks; Prophet and sktime are the other interval tools I tried.
