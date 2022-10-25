<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/US%20Bureau%20of%20Labor%20Statistics/US_Bureau_of_Labor_Statistics_Follow_CPI.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=US+Bureau+of+Labor+Statistics+-+Follow+CPI:+Error+short+description">🚨 Bug report</a>

**Tags:** #inflation #us #BLS #naas #scheduler #asset #snippet #automation #ai #analytics

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/j%C3%A9r%C3%A9my-ravenel-8a396910/)

The consumer price index (CPI) is the instrument used to measure inflation. It allows the estimation of the average variation between two given periods in the prices of products consumed by households. It is based on the observation of a fixed basket of goods updated every year.

The BLS provides us with a Python API: https://www.bls.gov/developers/api_python.html
Fortunately for us, the Python ecosystem is a supportive community of many open-source libraries, and thanks to data journalist Ben Welsh we have access to the CPI library with Python, which is a very nice wrapper around the BLS Python API that will save us a lot of work.

This template is inspired by the following article: https://pieriantraining.com/exploring-inflation-data-with-python/

## Input

### Setup libraries


```python
# Install CPI Python Package
!pip install cpi
```


```python
import pandas as pd
import cpi
import seaborn as sns
import matplotlib.pyplot as plt
import naas
from naas_drivers import plotly
```

## Model

### Update CPI latest data
BLS is constantly updating with the latest inflation information


```python
#this can take some time
cpi.update()
```

### Get monthly data


```python
cpi_items_df = cpi.series.get(seasonally_adjusted=False).to_dataframe()
cpi_items_df = cpi_items_df[cpi_items_df['period_type']=='monthly']
cpi_items_df['date'] = pd.to_datetime(cpi_items_df['date'])
cpi_items_df = cpi_items_df.set_index('date')
```

### Show dataframe


```python
df = cpi_items_df
df.head()
```

### Create plot


```python
fig = plt.figure(dpi=200)
df['value'].loc['2010':'2023'].plot(figsize=(10,4))
plt.title('Evolution of CPI')
plt.xlabel('Date')
plt.ylabel('CPI Value')
```

### Simulate 2% inflation YOY


```python
# Starting value
cpi_items_df['value'].loc['2010':'2023'].iloc[0]

start = cpi_items_df['value'].loc['2010':'2023'].iloc[0]

periods = len(cpi_items_df['value'].loc['2010':'2023'])//12
```


```python
def get_target_cpi(previous_cpi):
    return previous_cpi + 0.02*(previous_cpi)
```


```python
target_cpis = [start]
for year in range(0,periods):
    target_cpis.append(get_target_cpi(target_cpis[year]))
    
target_cpis
```


```python
dates = pd.date_range('2010-01-01','2023-01-01',periods=periods+1)
target_cpi_series = pd.Series(data =target_cpis, index= dates)
```


```python
fig = plt.figure(dpi=200)
cpi_items_df['value'].loc['2010':'2023'].plot(figsize=(10,6))
target_cpi_series.plot(ls='--')
plt.title('Evolution of CPI vs a 2% inflation rate yearly')
plt.xlabel('Date')
plt.ylabel('CPI Value');
```

## Output

### Save result in png


```python
plt.savefig("CPI.png")
```

### Save result in csv


```python
df.to_csv("CPI.csv", index=False)
```

### Share your output with naas


```python
naas.asset.add("CPI.csv")
naas.asset.add("CPI.png")

#-> Uncomment the line below to remove your asset
# naas.asset.delete()
```
