<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Follow_connections_from_profile.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Follow+connections+from+profile:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #network #connections #naas_drivers #analytics #csv #html #image #content #plotly

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import linkedin
import pandas as pd
from datetime import datetime
import naas
import plotly.graph_objects as go
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585
```

### Variables


```python
# Outputs
title = "LinkedIn - Connections"
name_output = "LinkedIn_connections"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

## Model

### Get connections from LinkedIn network
**Available columns :**
- FIRSTNAME : First name
- LASTNAME : Last name
- OCCUPATION : Text below the name in the profile page
- CREATED_AT : Date of connection between and your connection
- PROFILE_URL : Profile URL
- PROFILE_PICTURE : Profile picture URL
- PROFILE_ID : LinkedIn profile id
- PUBLIC_ID : LinkedIn public profile id


```python
df_connections = linkedin.connect(LI_AT, JSESSIONID).network.get_connections(limit=-1)
df_connections
```

### Prep trend data


```python
def get_trend(df,
              date_col_name='CREATED_AT',
              value_col_name="PROFILE_ID",
              date_order='asc'):
    
    # Format date
    df[date_col_name] = pd.to_datetime(df[date_col_name]).dt.strftime("%Y-%m-%d")
    df = df.groupby(date_col_name, as_index=False).agg({value_col_name: "count"})
    d = datetime.now().date()
    d2 = df.loc[df.index[0], date_col_name]
    idx = pd.date_range(d2, d, freq = "D")
    
    df.set_index(date_col_name, drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df[date_col_name] = pd.DatetimeIndex(df.index)
    
    # Calc sum cum
    df["VALUE_CUM"] = df.agg({value_col_name: "cumsum"})
    
    df["TEXT"] = (df['VALUE_CUM'].astype(str) + " as of " + df[date_col_name].dt.strftime("%Y-%m-%d") +
                  " (+" + df[value_col_name].astype(str) + " vs yesterday)")
    return df.reset_index(drop=True)

df_trend = get_trend(df_connections)
df_trend
```

### Create linechart


```python
def create_linechart(df, label, value, text, title):
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
#             line=dict(color="black"),
        )
    )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        title=f"<b>{title}</b><br><span style='font-size: 13px;'>{df.loc[df.index[-1], 'TEXT']}</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title="Date",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        yaxis_title='No. of connections',
        yaxis_title_font=dict(family="Arial", size=11, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_trend, "CREATED_AT", "VALUE_CUM", "TEXT", title)
```

## Output

### Save and share your csv file


```python
# Save your dataframe in CSV
df_trend.to_csv(csv_output, index=False)

# Share output with naas
csv_link = naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(csv_output)
```

### Save and share your graph in HTML


```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
html_link = naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```

### Save and share your graph in image


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
image_link = naas.asset.add(image_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
