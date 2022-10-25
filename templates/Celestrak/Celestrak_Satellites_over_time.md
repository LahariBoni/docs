<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Celestrak/Celestrak_Satellites_over_time.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Celestrak+-+Satellites+over+time:+Error+short+description">🚨 Bug report</a>

**Tags:** #celestrak #opendata #satellites #analytics #plotly

**Author:** [Dumorya](https://github.com/Dumorya)

We analyze the number of satellites in the space from the first launch to now, and the part of them which became inactive.
These data come from http://www.celestrak.com/. It provides free data as CSV files easily accessible.
The CSV file we got contains many data as the name of the satellite, its owner, launch site, id, apogee and many others.
What interested us the most were the status code, the launch date and the decay date, in order to create a graph with years in X axis and number of satellites in Y axis.

## Input

### Import librairies


```python
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import numpy as np
```

### Variables


```python
URL = 'http://www.celestrak.com/pub/satcat.csv'
```

## Model

### Recovery and processing of CSV data


```python
df = pd.read_csv(URL)
df = df[['OPS_STATUS_CODE', 'LAUNCH_DATE', 'DECAY_DATE']]
df['DECAY_DATE'] = df['DECAY_DATE'].mask(df['DECAY_DATE'].isnull(), '9999')
df['LAUNCH_DATE'] = df['LAUNCH_DATE'].str[:4].astype(int)
df['DECAY_DATE'] = df['DECAY_DATE'].str[:4].astype(int)
years = df['LAUNCH_DATE'].unique()
inactive = list()
active = list()

for year in years:
    active.append(len(df[
        ((df['OPS_STATUS_CODE'].isin(['+', 'P', 'B', 'S', 'X'])) & (df['LAUNCH_DATE'] <= year))
        | ((df['DECAY_DATE'] > year) & (df['OPS_STATUS_CODE'] == 'D') & (df['LAUNCH_DATE'] <= year))
    ].index))
    
    inactive.append(len(df[
        ((df['OPS_STATUS_CODE'] == 'D') & (df['DECAY_DATE'] <= year))
        | ((df['OPS_STATUS_CODE'] == '-') & (df['LAUNCH_DATE'] <= year) )
    ].index))
    
```

## Output

### Display plot of the number of satellites in space over time


```python
fig = go.Figure(data=[
    go.Bar(name='Inactive satellites', x=years, y=inactive),
    go.Bar(name='Active satellites', x=years, y=active)
])
# Change the bar mode
fig.update_layout(
    title="Number of satellites in space over time",
    xaxis_title="Years",
    yaxis_title="Number of satellites",
    barmode='stack'
)
fig.show()
```

Source: http://www.celestrak.com/pub/satcat.csv

### Display the percentage of inactive VS active satellites from 1957 to now


```python
labels = years
widths = [100/len(years) for year in years]

active_percentage = list()
inactive_percentage = list()

for index, _ in np.ndenumerate(active):
    total = active[index[0]] + inactive[index[0]]
    active_percentage.append(active[index[0]]/total*100)
    inactive_percentage.append(inactive[index[0]]/total*100)

data = {
    "Inactive": inactive_percentage,
    "Active": active_percentage
}

fig = go.Figure()
for key in data:
    fig.add_trace(go.Bar(
        name=key,
        y=data[key],
        x=years,
        offset=0
    ))

fig.update_xaxes(range=[years[0],years[-1]])
fig.update_yaxes(range=[0,100])

fig.update_layout(
    title_text="Percentage of inactive VS active satellites from 1957 to now",
    barmode="stack",
    uniformtext=dict(mode="hide", minsize=10),
)
```
