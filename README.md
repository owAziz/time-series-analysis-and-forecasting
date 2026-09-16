# I-94 Traffic Forecasting

Abdulaziz AlAbdulkareem  
SDAIA Academy — Time Series Analysis and Forecasting  
Cohort: 13–15 September 2026

<a href="https://colab.research.google.com/github/owAziz/time-series-analysis-and-forecasting/blob/main/capstone.ipynb" target="_blank" rel="noopener"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"></a>

I forecasted the next 24 hours of westbound traffic on I-94 (MN DoT station 301, between Minneapolis and St Paul) and attached an 80% prediction interval I can actually defend.

I used this series because it is hourly transport data with a clear daily rush-hour cycle, a weekly weekday/weekend split, holidays, and enough history for LightGBM after cutting to a contiguous 2018 window. Weather goes in only as a lag, so the forecast does not pretend I already know tomorrow’s rain.

Dataset: [UCI Metro Interstate Traffic Volume](https://archive.ics.uci.edu/dataset/492/metro+interstate+traffic+volume).

## Run it

Open `capstone.ipynb` from the badge, or clone the repo and run the notebook locally. The first code cell installs packages and pulls `metrics.py` / `backtest.py` from the [course repo](https://github.com/MohammadYusif/time-series-forecasting-ai-systems). No API key.

Course site: [Time Series Forecasting for AI Systems](https://mohammadyusif.github.io/time-series-forecasting-ai-systems/)  
Academy: [SDAIA Academy](https://github.com/SDAIAAcademy)

## What’s in the notebook

1. Decomposition, ACF/PACF, ADF
2. Holt-Winters with a Ljung-Box residual check
3. LightGBM with lag, rolling, and calendar features
4. Expanding-window backtest, 4 folds of 24 hours
5. MAE, RMSE, and MASE
6. Conformal 80% intervals (coverage and width)
7. Which model I would actually use, and why

Notes on leakage, the backtest, and the interval are in [TECHNICAL.md](TECHNICAL.md).
