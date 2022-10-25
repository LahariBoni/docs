<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WorldBank/WorldBank_Richest_countries_top10.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WorldBank+-+Richest+countries+top10:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldbank #opendata #snippet #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

**Goal**

Top 10 richest regions & countries in GDP per capita (wealth created per habitant) and in GDP current (total wealth created by the country)

**Data**

GDP CURRENT & GDP PER CAPITA by countries, agregated by region

**Sources**

* World Bank national accounts data
* OECD National Accounts data files 

**Notes**

The top 10 for GDP current is including the G8, should the European Union be included in this ranking, it would come up 2nd biggest economy after the USA. 

In the top 10 for GDP per capita, the ranking include smaller countries, only the USA remains in this ranking from the GDP current ranking.


**Pitch**

https://drive.google.com/file/d/1wGo9aI6mXS_2AbmnNlLoSJ9bGxkXmDhq/view

## Input

### Import libraries


```python
import pandas as pd
from pandas_datareader import wb
import plotly.graph_objects as go
```

## Model

### Get the association between the country and the region


```python
pd.options.display.float_format = '{: .0f}'.format
countries = wb.get_countries()
countries = countries[['name', 'region']]
countries
```

### Get indicators



```python
indicators = wb.download(indicator=['NY.GDP.PCAP.CD', 'NY.GDP.MKTP.CD'], country='all', start=2018, end=2018)
indicators = indicators.reset_index()
indicators = indicators[['country', 'NY.GDP.PCAP.CD', 'NY.GDP.MKTP.CD']]
indicators.columns = ['country', 'GDP_PER_CAPITA', 'CURRENT_GDP']
indicators
```

### Format a master table

1. Associate countries with regions
1. Clean up the data
1. Group rows by columns 


```python
master_table = pd.merge(indicators, countries, left_on='country', right_on='name')

master_table = master_table[master_table['region'] != 'Aggregates']
master_table = master_table[(master_table['GDP_PER_CAPITA'] > 0) | (master_table['CURRENT_GDP'] > 0)]
master_table = master_table.fillna(0)

master_table = pd.melt(master_table, id_vars=['region', 'country'], value_vars=['GDP_PER_CAPITA', 'CURRENT_GDP'], var_name='INDICATOR', value_name='VALUE')
master_table = master_table.set_index(['region', 'country', 'INDICATOR'])
master_table = master_table.sort_index()

master_table
```

## Output

### Visualize data with a chart


```python
table = master_table.reset_index()
gdp_per_capita_per_region = table[table['INDICATOR'] == 'GDP_PER_CAPITA'][['region', 'VALUE']].groupby('region').mean().sort_values('VALUE', ascending=False)
current_gdp_per_region = table[table['INDICATOR'] == 'CURRENT_GDP'][['region', 'VALUE']].groupby('region').mean().sort_values('VALUE', ascending=False)
gdp_per_capita_per_country = table[table['INDICATOR'] == 'GDP_PER_CAPITA'][['country', 'VALUE']].sort_values('VALUE', ascending=False).head(10)
current_gdp_per_country = table[table['INDICATOR'] == 'CURRENT_GDP'][['country', 'VALUE']].sort_values('VALUE', ascending=False).head(10)

data = [
  go.Bar(x=gdp_per_capita_per_region.index, y=gdp_per_capita_per_region['VALUE'], text=gdp_per_capita_per_region['VALUE'], textposition='outside'),
  go.Bar(x=current_gdp_per_region.index, y=current_gdp_per_region['VALUE'], text=current_gdp_per_region['VALUE'], textposition='outside', visible=False),
  go.Bar(x=gdp_per_capita_per_country['country'], y=gdp_per_capita_per_country['VALUE'], text=gdp_per_capita_per_country['VALUE'], textposition='outside', visible=False),
  go.Bar(x=current_gdp_per_country['country'], y=current_gdp_per_country['VALUE'], text=current_gdp_per_country['VALUE'], textposition='outside', visible=False),
]

layout = go.Layout(
  title='Top 10 richest regions & countries',
  margin = dict(t = 60, b = 150),
  updatemenus=list([
    dict(showactive=True, type="buttons", active=0, buttons=[
      {'label': 'GDP / Capita per region', 'method': 'update', 'args': [{'visible': [True, False, False, False]}]},
      {'label': 'Current GDP per region', 'method': 'update', 'args': [{'visible': [False, True, False, False]}]},
      {'label': 'GDP / Capita per country', 'method': 'update', 'args': [{'visible': [False, False, True, False]}]},
      {'label': 'Current GDP per country', 'method': 'update', 'args': [{'visible': [False, False, False, True]}]}
    ])
  ]),
  annotations=[dict(
    text = 'Updated in 2018 from The World Bank',
    showarrow = False,
    xref = 'paper', x = 1,
    yref = 'paper', y = -0.4)]
)

go.Figure(data, layout)
```
