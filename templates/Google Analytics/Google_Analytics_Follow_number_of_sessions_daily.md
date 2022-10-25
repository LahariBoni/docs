<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Analytics/Google_Analytics_Follow_number_of_sessions_daily.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Analytics+-+Follow+number+of+sessions+daily:+Error+short+description">🚨 Bug report</a>

**Tags:** #googleanalytics #analytics #marketing #csv #html #plotly #image #png

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import library


```python
import plotly.graph_objects as go
import naas
import datetime
import pandas as pd
from naas_drivers import googleanalytics
```

### Setup Google Analytics

👉 Create your own <a href="">Google API JSON credential</a>


```python
# Get your credential from Google Cloud Platform
json_path = 'naas-googleanalytics.json'

# Get view id from google analytics
view_id = "228952707"

# Setup your data parameters
dimensions = "daily" #hourly, daily, weekly, monthly
start_date = "30daysAgo" #XdaysAgo or date in ISO format %Y-%m-%d
end_date = "today" #Today or date in ISO format %Y-%m-%d
```

### Setup Outputs


```python
# Chart title
title = "Sessions"

# Outputs path
name_output = f"Google_Analytics_sessions_{dimensions}"
csv_output = f"{name_output}.csv"
image_output = f"{name_output}.png"
html_output = f"{name_output}.html"
```

## Model

### Get trend


```python
df = googleanalytics.connect(json_path, view_id).sessions.get_trend(dimensions, start_date, end_date)
df
```

### Plotting linechart


```python
def create_linechart(df: pd.DataFrame, label, value, varv, varp, title):
    """
    Plot linechart as an area chart in Plotly.
    """
    # Prep data
    df["VALUE_D"] = df[value].map("{:,.0f}".format).str.replace(",", " ")
    df["VARV_D"] = df[varv].map("{:,.0f}".format).str.replace(",", " ")
    df.loc[df[varv] > 0, "VARV_D"] = "+" + df["VARV_D"]
    df["VARP_D"] = df[varp].map("{:,.0%}".format).str.replace(",", " ")
    df.loc[df[varp] > 0, "VARP_D"] = "+" + df["VARP_D"]    

    # Create hovertext
    df["TEXT"] = (f"<b>{title} as of " + df[label].astype(str) + " : " + df["VALUE_D"] + "</b><br><span style='font-size: 13px;'>" + df["VARP_D"] + " vs last value (" + df["VARV_D"] + ")</span>")
    
    # Get subtitle
    title_display = df.loc[df.index[-1], "TEXT"] 
    
    # Create data
    data = go.Scatter(
        x=df[label],
        y=df[value],
        stackgroup="one",
        text=df["TEXT"],
        hoverinfo="text",
    )
    
    # Create layout
    layout = go.Layout(
        title=title_display,
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        margin_pad=10,
    )
    fig = go.Figure(data=data, layout=layout)
    fig.update_traces(mode='lines+markers')
    return fig

fig = create_linechart(df, "DATE", "VALUE", "VARV", "VARP", title)
fig
```

## Output

### Save and share csv


```python
df.to_csv(csv_output, index=False)

# Share with naas
#-> Uncomment the line below (by removing the hashtag) to share your asset with naas
# naas.asset.add(csv_output)

#-> Uncomment the line below (by removing the hashtag)  to delete your asset
# naas.asset.delete(csv_output)
```

### Export and share graph in HTML


```python
fig.write_html(html_output)

# Share with naas
#-> Uncomment the line below (by removing the hashtag) to share your asset with naas
# naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below (by removing the hashtag)  to delete your asset
# naas.asset.delete(html_output)
```

### Export and share graph in PNG


```python
fig.write_image(image_output)

# Share with naas
#-> Uncomment the line below (by removing the hashtag) to share your asset with naas
# naas.asset.add(image_output)

#-> Uncomment the line below (by removing the hashtag)  to delete your asset
# naas.asset.delete(image_output)
```
