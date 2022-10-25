<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WorldBank/WorldBank_GDP_per_country_and_evolution.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WorldBank+-+GDP+per+country+and+evolution:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldbank #opendata #snippet #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

Objective : allows to visualize the distribution of GDP per capita and the GDP growth in the world. Click on the country on the map or select it to see the details info

Data :
GDP PER CAPITA (CURRENT US$)
GDP GROWTH (ANNUAL %)

by countries, agregated by region

Sources:

World Bank national accounts data,
OECD National Accounts data files.


Production : Team Denver 2020/04/20 (MyDigitalSchool)

**source des données:** data.worldbank.org




**Introduction**: https://drive.google.com/file/d/1kM7_P18bwEPrsZSk8YsvOdiuJyLN1_3H/view?usp=sharing

## Input

### Get the data

*Récupération des données sur le PIB par pays:* 
https://data.worldbank.org/indicator/NY.GDP.PCAP.CD

*Récupération des données sur l'évolution du PIB par an par pays:* 
https://data.worldbank.org/indicator/NY.GDP.PCAP.KD.ZG

### Import libraries


```python
import pandas as pd
import numpy as np
import plotly.graph_objects as go
```

## Model

### Data formatting


```python
from pandas_datareader import wb

indicators = wb.download(indicator=['NY.GDP.PCAP.CD', 'NY.GDP.PCAP.KD.ZG'], country='all', start=2013, end=2018)

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

## Output

### Display the plot with plotly


```python
# Variable à changer pour avoir les autres années
year = "2018"
master_year_table = master_table[master_table['YEAR'] == year]

GDP_GROWTH_PER_CAPITAL = "GDP GROWTH PER CAPITAL"
GDP_PER_CAPITAL = "GDP PER CAPITAL"

fig = go.Figure()

fig.add_trace(go.Choropleth(
    locations=master_year_table['ISO3C'],
    z = master_year_table['GDP_PER_CAPITAL'],
    colorscale = [(0,"black"), (0.01,"red"),(0.1,"yellow"),(0.3,"green"),(1,"green")],
    colorbar_title = "GDP PER CAPITAL",
    customdata = master_year_table['COUNTRY'],
    hovertemplate = '<b>%{customdata}: %{z:,.0f}</b><extra></extra>'
))

fig.add_trace(go.Choropleth(
    locations=master_year_table['ISO3C'],
    visible= False,
    z = master_year_table['GDP_GROWTH_PER_CAPITAL'],
    colorscale = [(0,"red"),(0.5,"red"),(0.75,"rgb(240,230,140)"), (1,"green")],
    colorbar_title = "GDP GROWTH PER CAPITAL",
    customdata = master_year_table['COUNTRY'],
    hovertemplate = '<b>%{customdata}: %{z:0.2f}%</b><extra></extra>'
))

fig.update_layout(
    autosize=False,
    width= 1600,
    height= 900,
    title=f"GDP per capital in {year}",
    title_x=0.5,
    updatemenus=[
        dict(
            type = "buttons",
            active=0,
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
            x=1,
            xanchor="right",
            y=1.1,
            yanchor="top"
        ),
    ]
)

fig.show()
```
