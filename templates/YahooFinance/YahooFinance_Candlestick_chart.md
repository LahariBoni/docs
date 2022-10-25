<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YahooFinance/YahooFinance_Candlestick_chart.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YahooFinance+-+Candlestick+chart:+Error+short+description">🚨 Bug report</a>

**Tags:** #yahoofinance #trading #yfin #investors #snippet #plotly

**Author:** [Carlo Occhiena](https://www.linkedin.com/in/carloocchiena/)

## Input

### Install and import libraries


```python
import datetime as dt
import pandas_datareader as pdr
try:
    import mplfinance as mpf
except:
    !pip install mplfinance
    import mplfinance as mpf
try:
    import yfinance as yfin
except:
    !pip install yfinance
    import yfinance as yfin
```

### Variables
datefrom, dateto, the list of cryptocurrencies you need.


```python
# update date range "start" as desired
start = dt.datetime(2021, 1, 1)
end = dt.datetime.now()
ticker = "ETH-USD"
```

## Model
candlestick chart, correlation heatmap, visual chart for all the currencies.


```python
# insert cryptoasset (in Yahoo format "NNN-CCC" es BTC-EUR or ETH-USD) here
yfin.pdr_override()
data = yfin.download(ticker, start, end)
data
```

## Output

### Create a candlestick chart for a given asset
👉 Completely customize the timeframe and asset you need


```python
mpf.plot(data, title=f"{ticker} trend",
         type="candle",
         volume=True,
         style="yahoo")
```


```python

```
