<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pandas/Pandas_Format_number_to_string.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pandas+-+Format+number+to+string:+Error+short+description">🚨 Bug report</a>

**Tags:** #pandas #dataframe #format #snippet #yahoofinance #naas_drivers #operations #jupyternotebooks

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

With this template, you can convert a series of number nicely using pandas.map

## Input

### Import libraries


```python
from naas_drivers import yahoofinance
```

### Input parameters


```python
ticker = "TSLA"
date_from = -100
date_to = 'today'
```

## Model

### Get dataframe from Yahoo Finance


```python
df_yahoo = yahoofinance.get(ticker,
                            date_from=-5)
df_yahoo
```

## Output

### Format number nicely
Get more information in the doc [here](https://pandas.pydata.org/docs/reference/api/pandas.Series.map.html)


```python
df_yahoo['VALUE'] = df_yahoo['Close'].map("€ {:,.2f}".format).str.replace(",", " ")
df_yahoo
```


```python

```
