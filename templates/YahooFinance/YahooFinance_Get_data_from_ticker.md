<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YahooFinance/YahooFinance_Get_data_from_ticker.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YahooFinance+-+Get+data+from+ticker:+Error+short+description">🚨 Bug report</a>

**Tags:** #yahoofinance #trading #naas_drivers #investors #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

With this template, you can get data from any ticker available in [Yahoo finance](https://finance.yahoo.com/quote/TSLA/).<br> 

## Input

### Import libraries


```python
from naas_drivers import yahoofinance
```

### Input parameters
👉 Here you can change the ticker, timeframe and add moving averages analysis


```python
ticker = "TSLA"
date_from = -100
date_to = 'today'
interval = '1d'
moving_averages = [20, 50]
```

## Model

### Get dataset from Yahoo Finance


```python
df_yahoo = yahoofinance.get(ticker,
                            date_from=date_from,
                            date_to=date_to,
                            interval=interval,
                            moving_averages=moving_averages)
```

## Output

### Display result


```python
df_yahoo
```
