# I-94 Traffic Forecasting

Abdulaziz AlAbdulkareem  
SDAIA Academy — Time Series Analysis and Forecasting  
13–15 September 2026

Forecast of the next 24 hours of westbound traffic on I-94 (MN DoT station 301, Minneapolis–St Paul).

I used the [UCI Metro Interstate Traffic Volume](https://archive.ics.uci.edu/dataset/492/metro+interstate+traffic+volume) series. It is hourly, has a clear rush-hour cycle and a weekday/weekend split, and 2018 is long enough to fit a tree model after cleaning. Weather is lagged so the forecast does not assume I already know tomorrow’s rain.

Run `capstone.ipynb` in Jupyter or Colab.

[SDAIA Academy](https://github.com/SDAIAAcademy)

More on the pipeline and leakage is in [TECHNICAL.md](TECHNICAL.md).
