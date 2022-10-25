<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WSR/WSR_Get_daily_Covid19_active_cases_trend_JHU.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WSR+-+Get+daily+Covid19+active+cases+trend+JHU:+Error+short+description">🚨 Bug report</a>

**Tags:** #wsr #covid #active-cases #plotly #opendata #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
import pandas as pd
from datetime import datetime
import plotly.graph_objects as go
import naas
```

### Variables


```python
# Input URLs of the raw csv dataset
urls = [
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv'
]

# Outputs
uid = "WSR_00002"
name_output = "Covid19_Activecases_trend"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
title = "COVID 19 - Active cases (in milions)"
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

### Get trend


```python
def get_trend(df_init):
    df = df_init.copy()
    # Filter
    df = df[(df["INDICATOR"] == "Active cases")]
    
    # Groupby date
    df = df.groupby(["DATE"], as_index=False).agg({"VALUE": 'sum'})
    
    # Calc variation
    for idx, row in df.iterrows():
        if idx == 0:
            value_n1 = 0
        else:
            value_n1 = df.loc[df.index[idx-1], "VALUE"]
        df.loc[df.index[idx], "VALUE_COMP"] = value_n1
    df["VARV"] = df["VALUE"] - df["VALUE_COMP"]
    df["VARP"] = df["VARV"] / abs(df["VALUE_COMP"])
    
    # Calc variation
    for idx, row in df.iterrows():
        if idx == 0:
            value_n1 = 0
        else:
            value_n1 = df.loc[df.index[idx-1], "VARP"]
        df.loc[df.index[idx], "VARP_COMP"] = value_n1
    df["VARP_VAR"] = df["VARP"] - df["VARP_COMP"]
    
    # Prep data
    df["VALUE_D"] = df["VALUE"].map("{:,.0f}".format).str.replace(",", " ")
    df["VARV_D"] = df["VARV"].map("{:,.0f}".format).str.replace(",", " ")
    df.loc[df["VARV"] > 0, "VARV_D"] = "+" + df["VARV_D"]
    df["VARP_D"] = df["VARP"].map("{:,.2%}".format).str.replace(",", " ")
    df.loc[df["VARP"] > 0, "VARP_D"] = "+" + df["VARP_D"]
    
    df["TEXT"] = (df['VALUE_D'] + " active cases as of " + df['DATE'].dt.strftime("%Y-%m-%d") + "<br>"
                  " (" + df['VARP_D'] + " vs yesterday)")
    return df

df_trend = get_trend(df_clean)
df_trend.tail(30)
```

### Create linechart


```python
def create_linechart(df, label, value, text):
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value],
            text=df[text],
            hoverinfo="text",
            mode="lines",
            line=dict(color="black"),
        )
    )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        title=title,
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title="Date",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        yaxis_title='No. of active cases',
        yaxis_title_font=dict(family="Arial", size=11, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_trend, "DATE", "VALUE", "TEXT")
```

## Output

### Save and share csv with naas


```python
# Save your dataframe in CSV
df.to_csv(csv_output, index=False)

# Share output with naas
naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(csv_output)
```

### Save and share image with naas


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
naas.asset.add(image_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```

### Save and share html with naas


```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```
