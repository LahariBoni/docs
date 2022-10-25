<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Analytics/Google_Analytics_Get_bounce_rate.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Analytics+-+Get+bounce+rate:+Error+short+description">🚨 Bug report</a>

**Tags:** #googleanalytics #bouncerate #plotly #linechart #naas_drivers #scheduler #asset #naas #marketing #analytics #automation #image #csv #html

**Author:** [Charles Demontigny](https://www.linkedin.com/in/charles-demontigny/)

Pre-requisite: Create your own <a href="">Google API JSON credential</a>

## Input

### Import library


```python
import pandas as pd
import plotly.graph_objects as go
import naas
from naas_drivers import googleanalytics
```

### Get your credential from Google Cloud Platform


```python
json_path = 'naas-googleanalytics.json'
```

### Get view id from google analytics


```python
view_id = "228952707"
```

### Setup your output paths


```python
csv_output = "googleanalytics_bounce_rate.csv"
html_output = "googleanalytics_bounce_rate.html"
```

### Schedule your notebook


```python
naas.scheduler.add(cron="0 8 * * *")
naas.dependency.add(json_path)

#-> Uncomment the line below (by removing the hashtag) to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Bounce Rate


```python
df_bounce_rate = googleanalytics.connect(json_path=json_path).views.get_bounce_rate(view_id=view_id)
df_bounce_rate
```

## Output

### Save dataframe in csv


```python
df_bounce_rate.to_csv(csv_output, index=False)
```

### Bounce Rate Plot


```python
def plot_bounce_rate(df: pd.DataFrame):
    """
    Plot bounce rate as an area chart in Plotly.
    """
    # Prep dataframe
    df["Date"] = pd.to_datetime(df['Year Month'] + "01")
    
    # Get total views
    value = "{:,.0%}".format(df["Bounce Rate"].mean())
    
    # Create data
    data = go.Scatter(
        x=df["Date"],
        y=df['Bounce Rate'],
        stackgroup="one"
    )
    
    # Create layout
    layout = go.Layout(
        yaxis={"tickformat": ',.0%'},
        title=f"<b>Bounce Rate</b><br><span style='font-size: 13px;'>Average bounce rate: {value}</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        yaxis_title="Bounce rate %",
        yaxis_title_font=dict(family="Arial", size=11, color="black"),
        xaxis_title="Mounths",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        margin_pad=10,
    )
    fig = go.Figure(data=data, layout=layout)
    fig.update_traces(mode='lines+markers')
    return fig

fig = plot_bounce_rate(df_bounce_rate)
fig
```

### Export and share graph


```python
# Export in HTML
fig.write_html(html_output)

# Shave with naas
#-> Uncomment the line below (by removing the hashtag) to share your asset with naas
# naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below (by removing the hashtag)  to delete your asset
# naas.asset.delete(html_output)
```
