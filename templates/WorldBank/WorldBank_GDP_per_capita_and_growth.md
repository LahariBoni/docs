<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WorldBank/WorldBank_GDP_per_capita_and_growth.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WorldBank+-+GDP+per+capita+and+growth:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldbank #opendata #snippet #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

**Objective** : allows to visualize a map of GDP per capita and the GDP growth of all the countries in the world. Click on the country on the map or select it to see the details info.

**Data** :
- GDP per capita (current US$)
- GDP growh (annual %)

**Source**: World Bank national accounts data.

## Input

### Import library


```python
import pandas as pd
import numpy as np
import plotly.graph_objects as go
from pandas_datareader import wb
from naas_drivers import plotly
```

## Model

### Get data from World Bank


```python
indicators = wb.download(indicator=['NY.GDP.PCAP.CD', 'NY.GDP.PCAP.KD.ZG'], country='all', start=2013, end=2021)

indicators = indicators.reset_index()
indicators = indicators[['country', 'year', 'NY.GDP.PCAP.CD', 'NY.GDP.PCAP.KD.ZG']]
indicators.columns = ['country', 'year', 'GDP_PER_CAPITAL', 'GDP_GROWTH_PER_CAPITAL']

indicators = indicators.fillna(0)

countries = wb.get_countries()
countries = countries[['name', 'region', 'iso3c']]

master_table = pd.merge(indicators, countries, left_on='country', right_on='name')

master_table = master_table[master_table['region'] != 'Aggregates']

master_table = master_table.drop(columns=['name'])

master_table = master_table.dropna()

# Création de l'ensemble final
xls_formatted = pd.DataFrame(columns=['COUNTRY', 'YEAR', 'GDP_PER_CAPITAL', 'GDP_GROWTH_PER_CAPITAL', 'REGION', 'ISO3C'])

for index, line in master_table.iterrows():
  xls_formatted = xls_formatted.append(
    {
        'COUNTRY': line['country'],
        'YEAR': line['year'],
        'GDP_PER_CAPITAL': line['GDP_PER_CAPITAL'],
        'GDP_GROWTH_PER_CAPITAL': line['GDP_GROWTH_PER_CAPITAL'],
        'REGION': line['region'],
        'ISO3C': line['iso3c'],
    }, ignore_index=True
  )

master_table = xls_formatted

master_table
```

### Choose the year to display


```python
year = "2019"
```

### Create mapchart


```python
master_year_table = master_table[master_table['YEAR'] == year]

GDP_GROWTH_PER_CAPITAL = "GDP growth per capita"
GDP_PER_CAPITAL = "GDP per capita"

fig = go.Figure()

fig.layout = go.Layout(
    #width=500,
    #height=300,
    annotations=[
        go.layout.Annotation(
            showarrow=False,
            text='Source: World Bank',
            xanchor='right',
            x=1,
            yanchor='top',
            y=-0.05
        )])

fig.add_trace(go.Choropleth(
    locations=master_year_table['ISO3C'],
    z = master_year_table['GDP_PER_CAPITAL'],
    colorscale = [(0,"#053D8B"),(0.25,"#1164B0"),(0.5,"#298BC8"),(0.75,"#3FB7DB"), (1,"#5CD5E8")],
    colorbar_title = "Value",
    customdata = master_year_table['COUNTRY'],
    hovertemplate = '<b>%{customdata}: %{z:,.0f}</b><extra></extra>'
))

fig.add_trace(go.Choropleth(
    locations=master_year_table['ISO3C'],
    visible= False,
    z = master_year_table['GDP_GROWTH_PER_CAPITAL'],
    colorscale = [(0,"#053D8B"),(0.25,"#1164B0"),(0.5,"#298BC8"),(0.75,"#3FB7DB"), (1,"#5CD5E8")],
    colorbar_title = "Growth ",
    customdata = master_year_table['COUNTRY'],
    hovertemplate = '<b>%{customdata}: %{z:0.2f}%</b><extra></extra>'
))

fig.update_layout(
    autosize=True,
    width= 900,
    #height= 900,
    title=f"GDP per capital in {year}",
    title_x=0.5,
    title_y=0.95,
    updatemenus=[
        dict(
            type = "buttons",
            active=0,
            direction = "left",
            buttons=list([
                dict(
                    args=[{"visible": [True, False]}, {"title": f"{GDP_PER_CAPITAL} in {year}"}],
                    label=GDP_PER_CAPITAL,
                    method="update"
                ),
                dict(
                    args=[{"visible": [False, True]}, {"title": f"{GDP_GROWTH_PER_CAPITAL} in {year}"}],
                    label=GDP_GROWTH_PER_CAPITAL,
                    method="update"
                )
            ]),
            showactive=True,
            x=0.32,
            xanchor="left",
            y=1.2,
            yanchor="top"
        ),
    ]
)

fig.show()
```

## Output

### Export chart


```python
plotly.export(fig, "GDP.png", css=None)
plotly.export(fig, "GDP.html", css=None)
```
