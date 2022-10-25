<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WSR/WSR_Get_daily_Covid19_active_cases_worldmap_JHU.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WSR+-+Get+daily+Covid19+active+cases+worldmap+JHU:+Error+short+description">🚨 Bug report</a>

**Tags:** #wsr #covid #active-cases #analytics #plotly #automation #naas #opendata #image

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
import pandas as pd
from datetime import datetime
try:
    from dataprep.clean import clean_country
except:
    !pip install dataprep --user
    from dataprep.clean import clean_country
import plotly.graph_objects as go
import naas
```

### Setup chart title


```python
title = "COVID 19 - Active cases (in milions)"
```

### Variables


```python
# Input URLs of the raw csv dataset
urls = [
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv'
]

# Output paths
output_image = f"{title}.png"
output_html = f"{title}.html"
```

### Schedule your automation


```python
# Schedule your job everyday at 8:00 AM (NB: you can choose the time of your scheduling bot)
naas.scheduler.add(cron="0 8 * * *")

#-> Uncomment the line below (by removing the hashtag) to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get data from JHU


```python
def get_data_url(urls):
    df = pd.DataFrame()
    for url in urls:
        tmp_df = pd.read_csv(url)
        tmp_df["Indicator"] = url.split("/time_series_covid19_")[-1].split("_global.csv")[0].capitalize()
        df = pd.concat([df, tmp_df])
    return df

df_init = get_data_url(urls)
df_init
```

### Get all data from JHU


```python
def get_all_data(df_init):
    df = df_init.copy()
    # Cleaning
    df = df.drop("Province/State", axis=1)
    
    # Melt data
    df = pd.melt(df,
                 id_vars=["Country/Region", "Lat", "Long", "Indicator"],
                 var_name="Date",
                 value_name="Value").fillna(0)
    df["Date"] = pd.to_datetime(df["Date"])
    
    # Calc active cases
    df_active = df.copy()
    df_active.loc[df_active["Indicator"].isin(["Deaths", "Recovered"]), "Value"] = df_active["Value"] * (-1)
    df_active["Indicator"] = "Active cases"
    
    # Concat data
    df = pd.concat([df, df_active])
    
    # Group by country/region
    to_group = ["Country/Region", "Lat", "Long", "Indicator", "Date"]
    df = df.groupby(to_group, as_index=False).agg({"Value": "sum"})
    
    # Cleaning
    df = df.rename(columns={"Country/Region": "COUNTRY"})
    df.columns = df.columns.str.upper()
    return df.reset_index(drop=True)

df_clean = get_all_data(df_init)
df_clean
```

### Prep data for worldmap


```python
def prep_data(df_init):
    df = df_init.copy()
    # Filter
    date_max = df["DATE"].max()
    df = df[
        (df["INDICATOR"] == "Active cases") & 
        (df["DATE"] == date_max)
    ].reset_index(drop=True)


    # Clean country
    df = clean_country(df, 'COUNTRY', output_format='alpha-3').dropna()
    df = df.rename(columns={'COUNTRY_clean': 'COUNTRY_ISO'})
    return df.reset_index(drop=True)

df_worldmap = prep_data(df_clean)
df_worldmap
```

### Create worldmap


```python
def create_worldmap(df):
    fig = go.Figure()

    fig = go.Figure(data=go.Choropleth(
        locations=df['COUNTRY_ISO'],
        z=df['VALUE'],
        text=df["COUNTRY"] + ": " + df['VALUE'].map("{:,.0f}".format).str.replace(",", " ") + " active cases",
        hoverinfo="text",
        colorscale='Blues',
        autocolorscale=False,
        reversescale=False,
        marker_line_color='darkgray',
        marker_line_width=0.5,
        colorbar_tickprefix='',
        colorbar_title='Active cases',
    ))

    fig.update_layout(
        title=title,
        plot_bgcolor="#ffffff",
        legend_x=1,
        geo=dict(
            showframe=False,
            showcoastlines=False,
            #projection_type='equirectangular'
        ),
        dragmode= False,
        width=1200,
        height=800,

    )
    config = {'displayModeBar': False}
    fig.show(config=config)
    return fig

fig = create_worldmap(df_worldmap)
```

## Output

### Export in PNG and HTML


```python
fig.write_image(output_image, width=1200)
fig.write_html(output_html)
```

### Generate shareable assets


```python
link_image = naas.asset.add(output_image)
link_html = naas.asset.add(output_html, {"inline":True})

#-> Uncomment the line below to remove your assets
# naas.asset.delete(output_image)
# naas.asset.delete(output_html)
```
