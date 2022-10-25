<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Excel/Excel_Save_file.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Excel+-+Save+file:+Error+short+description">🚨 Bug report</a>

**Tags:** #excel #pandas #save #opendata #yahoofinance #naas_drivers #finance #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
import pandas as pd
from naas_drivers import yahoofinance
```

### Variables


```python
# Input yahoo
ticker = "TSLA"
date_from = -100
date_to = 'today'
interval = '1d'
moving_averages = [20, 50]

# Output
excel_file_path = f"{ticker}.xlsx"
```

## Model

### Read excel


```python
df_yahoo = yahoofinance.get(ticker,
                            date_from=date_from,
                            date_to=date_to,
                            interval=interval,
                            moving_averages=moving_averages)
```

## Output

### Display result

You want to add more parameters ?<br>
👉 Check out the pandas documentation <a href="https://pandas.pydata.org/docs/reference/api/pandas.read_excel.html">here</a>.


```python
df_yahoo.to_excel(excel_file_path)
print(f'💾 Excel '{excel_file_path}' successfully saved in Naas.')
```
