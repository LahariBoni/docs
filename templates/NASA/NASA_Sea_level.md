<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/NASA/NASA_Sea_level.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=NASA+-+Sea+level:+Error+short+description">🚨 Bug report</a>

**Tags:** #nasa #naas #opendata #analytics #plotly

**Author:** [Colyn TIDMAN](https://www.linkedin.com/in/colyntidman/), [Dylan PICHON](https://www.linkedin.com/in/dylan-pichon/)

Sea level rise is caused primarily by two factors related to global warming: the added water from melting ice sheets and glaciers and the expansion of seawater as it warms. The first graph tracks the change in sea level since 1993 as observed by satellites.

The second graph, derived from coastal tide gauge and satellite data, shows how much sea level changed from about 1900 to 2018. Items with pluses (+) are factors that cause global mean sea level to increase, while minuses (-) are variables that cause sea levels to decrease. These items are displayed at the time they were affecting sea level.

The data shown are the latest available, with a four- to five-month lag needed for processing.

* You now need to create an Earthdata account to access NASA's sea level data. Register for free by clicking on 'Get data : http'. Once logged in you will access the data.

Website : https://climate.nasa.gov/vital-signs/sea-level/

Data source: Satellite sea level observations.
Credit: NASA's Goddard Space Flight Center

## Input

### Import libraries


```python
import pandas
import plotly.graph_objects as go
```

### Path of the source

Data source : nasa_sea_levels.txt downloaded earlier


```python
uri_nasa_sea_level = "nasa-sea-level-data.txt"
```

## Model

### Read the csv and create the table


```python
df = pandas.read_csv(uri_nasa_sea_level, engine="python", comment='HDR',delim_whitespace=True, names=["A","B","Year + Fraction","D","E","F","G", "H","I","J","K","Smoothed GMSL (mm)",])

df.head(10)
```

Now lets only get the information we want and convert year + fraction to date


```python
new_df = pandas.DataFrame(df, columns=['Year + Fraction', 'Smoothed GMSL (mm)'])

dates = []
values = []

ref = 0

for i, row in new_df.iterrows():
    #date formating
    date_split = str(row['Year + Fraction']).split('.')
    year = date_split[0]
    fraction = '0.' + date_split[1]
    float_fraction = float(fraction)
    date = year + "-1-1"
    date_delta = 365 * float_fraction
    value = pandas.to_datetime(date) + pandas.to_timedelta(date_delta, unit='D')
    dates.append(value)
    
    #value formating
    #to stay inline with the graph visible on nasa's website, we need to have 0 as our first value
    if i == 0:
       ref = row['Smoothed GMSL (mm)'] 
    
    val = row['Smoothed GMSL (mm)'] - ref 
    values.append(val)
    
    
    

new_df['Date'] = dates
new_df['Value'] = values
    
new_df.head()
```

## Output

### Land-Ocean Temperature Index - Visualization


```python
fig = go.Figure(layout_title="<b>Sea Level variation since 1993 (mm)</b>")
fig.add_trace(go.Scatter(
    x = new_df["Date"],
    y = new_df["Value"],
    name="Delta",
))

fig.update_layout(
    autosize=False,
    width=1300,
    height=700,
    plot_bgcolor='rgb(250,250,250)',
)


fig.add_annotation(y=6, x='2020-1-1',
            text="Data source: Satellite sea level observations.<br> Credit: NASA's Goddard Space Flight Center",
            showarrow=False,
            )

fig.update_yaxes(title_text="Sea Height Variation (mm)")
fig.update_xaxes(title_text="Year", tickangle=60)
fig.add_hline(y=0.0)
fig.update_layout(title_x=0.5)

fig.show()
```
