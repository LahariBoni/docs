<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/CCXT/CCXT_Calculate_Support_and_Resistance.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=CCXT+-+Calculate+Support+and+Resistance:+Error+short+description">🚨 Bug report</a>

**Tags:** #ccxt #bitcoin #trading #investors #analytics #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input


```python
!pip install trendln matplotlib==3.1.3 --user
```


```python
import naas
import ccxt
import pandas as pd
from datetime import datetime
import naas_drivers
import trendln
import plotly.tools as tls
import plotly.graph_objects as go
```

### Setup Binance
👉 <a href='https://www.binance.com/en/support/faq/360002502072'>How to create API ?</a>


```python
binance_api = ""
binance_secret = ""
```

### Variables


```python
symbol = 'BTC/USDT'
limit = 180
timeframe = '4h'
```

## Model

### Get data


```python
binance = ccxt.binance({
    'apiKey': binance_api,
    'secret': binance_secret
}) 

data = binance.fetch_ohlcv(symbol=symbol,
                           limit=limit,
                           timeframe=timeframe)
```

### Data cleaning


```python
df = pd.DataFrame(data, columns=["Date", "Open", "High", "Low", "Close", "Volume"])
df['Date'] = [datetime.fromtimestamp(float(time)/1000) for time in df['Date']]
df
```

## Output

### Plotting figure


```python
fig = trendln.plot_support_resistance(
    df[-1000:].Close, #as per h for calc_support_resistance
    xformatter = None, #x-axis data formatter turning numeric indexes to display output
      # e.g. ticker.FuncFormatter(func) otherwise just display numeric indexes
    numbest = 1, #number of best support and best resistance lines to display
    fromwindows = True, #draw numbest best from each window, otherwise draw numbest across whole range
    pctbound = 0.1, # bound trend line based on this maximum percentage of the data range above the high or below the low
    extmethod = trendln.METHOD_NUMDIFF,
    method=trendln.METHOD_PROBHOUGH,
    window=125,
    errpct = 0.005,
    hough_prob_iter=50,
    sortError=False,
    accuracy=1)
```


```python
plotly_fig = tls.mpl_to_plotly(fig)
```


```python
layout = dict(
    dragmode="pan",
    xaxis_rangeslider_visible=False,
    showlegend=True,
)
new_data = list(plotly_fig.data)
new_data.pop(2)
new_data.pop(2)
new_data.pop(1)
new_data.pop(1)
fig = go.Figure(data=new_data, layout=layout)
fig
```
