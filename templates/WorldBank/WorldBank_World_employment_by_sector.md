<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WorldBank/WorldBank_World_employment_by_sector.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WorldBank+-+World+employment+by+sector:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldbank #opendata #snippet #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

**Objective**

This graph compares the world distribution of employment by sector with the country distribution. Select the country to visualize which sector is dominant.

Data
by countries, by region

Source
International Labour Organization, ILOSTAT database.

Video : 
https://drive.google.com/file/d/1AISsc-lc4-94Dj7LOQKBj3L7VgChkejG/view

## Input

### Import library


```python
import math
import pandas as pd
from datetime import datetime
from plotly.offline import iplot, plot, download_plotlyjs, init_notebook_mode
import plotly.graph_objects as go
from plotly.subplots import make_subplots
```

### Variables


```python
URL = "https://www.ilo.org/ilostat-files/Documents/Excel/MBI_33_EN.xlsx"

SELECTED_YEAR = 2019
```

## Model

### Retrieving data


```python
retrieved_data = pd.read_excel(URL, sep='\t', parse_dates=[0],
                     names=['COUNTRY','b','GENDER','YEAR','e','AGRICULTURE - FORESTRY - FISHING','MINING - QUARRYING','MANUFACTURING','UTILITIES','CONSTRUCTION',
                            'WHOLESALE - RETAIL - REPAIR VEHICLES','TRANSPORT - STORAGE - COMMUNICATION','ACCOMODATION - FOOD SERVICES','FINANCE - INSURANCE',
                            'REAL ESTATE - BUSINESS - ADMINISTRATION','PUBLIC ADMINISTRATION - DEFENCE - SOCIAL SECURITY','EDUCATION','HUMAN HEALTH - SOCIAL WORK',
                            'OTHER SERVICES','t','u','v','w','x','y','z','aa','ab','ac','ad','ae','af','ag','ah'])


```

### Data selection


```python
# Select only rows with data
data = retrieved_data.drop([0, 1, 2, 3, 4], axis = 0)

# Select rows we want ("Total" of gender and no male nor female)
data = data[data['GENDER'] == "Total"]

# Select by year
data = data[data['YEAR'] == SELECTED_YEAR]

# Select only columns with interest
df = data[['COUNTRY', 'AGRICULTURE - FORESTRY - FISHING','MINING - QUARRYING','MANUFACTURING','UTILITIES','CONSTRUCTION',
              'WHOLESALE - RETAIL - REPAIR VEHICLES','TRANSPORT - STORAGE - COMMUNICATION','ACCOMODATION - FOOD SERVICES','FINANCE - INSURANCE',
              'REAL ESTATE - BUSINESS - ADMINISTRATION','PUBLIC ADMINISTRATION - DEFENCE - SOCIAL SECURITY','EDUCATION','HUMAN HEALTH - SOCIAL WORK',
              'OTHER SERVICES']]
               
df = df.set_index('COUNTRY')
df
```

## Output

### Data formatting


```python
# Initialization of variables and functions

SECTORS = []
for col in df.columns :
  SECTORS.append(col)

indexstock = ['World', "France"]

#***********************************************************************************
# Initialization of graphs

fig = make_subplots(rows=2, cols=2, 
                    specs=[[{'type': 'polar'}, {'type': 'polar'}],[{"colspan": 2}, None]],
                    shared_xaxes=True, shared_yaxes=False, row_heights=[0.3, 0.7],
                    vertical_spacing=0.11, horizontal_spacing=0.15)

#***********************************************************************************
# Add polar graphs

fig.add_trace(go.Scatterpolar(r = df.loc[indexstock[0]], theta = SECTORS, fill = 'toself', name = indexstock[0], 
                                  marker_color='mediumpurple'), row = 1, col = 1)
fig.add_trace(go.Scatterpolar(r = df.loc[indexstock[1]], theta = SECTORS, fill = 'toself', name = indexstock[1], 
                                  marker_color='indianred') ,row = 1, col = 2)

#***********************************************************************************
# Horizontal group bar graph 

# print(df.loc['World'])
fig.add_trace(go.Bar(x=df.loc[indexstock[0]], y=SECTORS, orientation='h', name=indexstock[0], text=df.loc[indexstock[0]], 
                     textposition='auto', marker_color='mediumpurple'), row = 2, col = 1)
fig.add_trace(go.Bar(x=df.loc[indexstock[1]], y=SECTORS, orientation='h', name=indexstock[1], text=df.loc[indexstock[1]], 
                     textposition='auto', marker_color='indianred'), row = 2, col = 1)

#***********************************************************************************
# Setting layout

fig.update_layout(title_text="Differences of repartition of employement by country and sector au " + str(datetime.today()) + " (en %)",
                  title_x=0.5, width=1600, height=1000, 
                  showlegend=True, legend=dict(x=-.2, y=0.8),
                  polar=dict(radialaxis=dict(visible=True)),
                  barmode='group')

#***********************************************************************************
# Creationg buttons

buttons_country_1 = []
for index in df.index:
  buttons_country_1.append(dict(method='restyle', label=index, args=[{'r':[df.loc[index]], 'x':[df.loc[index]], 'name':[index, index], 'text':[df.loc[index]]}, [0, 2]]))
buttons_country_2 = []
for index in df.index:
  buttons_country_2.append(dict(method='restyle', label=index, args=[{'r':[df.loc[index]], 'x':[df.loc[index]], 'name':[index, index], 'text':[df.loc[index]]}, [1, 3]]))

fig.update_layout(updatemenus=[dict(buttons=buttons_country_1, direction="down", pad={"r": 1, "t": 1}, showactive=True, x=0.04, xanchor="left", y=0.69, yanchor="top"),
                              dict(buttons=buttons_country_2, direction="down", pad={"r": 1, "t": 1}, showactive=True, x=0.6, xanchor="left", y=0.69, yanchor="top")])

#***********************************************************************************
# Display graph

fig.show()

#***********************************************************************************
# Export as HTML file

Tickets_plot = fig
plot(Tickets_plot, filename="/content/gdrive/My Drive/datamining/employement_by_sector_and_country.html", auto_open=False)


```
