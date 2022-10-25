<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Analytics/Google_Analytics_Get_pageview_ranking.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Analytics+-+Get+pageview+ranking:+Error+short+description">🚨 Bug report</a>

**Tags:** #googleanalytics #pageviews #plotly #horizontalbarchart #naas_drivers #scheduler #asset #naas #marketing #analytics #automation #image #csv #html

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
csv_output = "googleanalytics_pages_views.csv"
html_output = "googleanalytics_pages_views.html"
```

### Schedule your notebook


```python
naas.scheduler.add(cron="0 8 * * *")
naas.dependency.add(json_path)

#-> Uncomment the line below (by removing the hashtag) to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Ranking: Most visited web pages


```python
df_pageview = googleanalytics.connect(json_path=json_path).views.get_pageview(view_id)
df_pageview
```

## Output

### Save dataframe in csv


```python
df_pageview.to_csv(csv_output, index=False)
```

### Plotting horizontal barchart


```python
def plot_pageview(df: pd.DataFrame):
    """
    Plot PageView in Plotly.
    """
    # Prep dataframe
    df.loc[df.Pages == "/", "Pages"] = "landing"
    df.loc[df.Pages != "landing", "Pages"] = df.Pages.str[1:]
    
    # Get total views
    value = "{:,.0f}".format(df["Pageview"].sum()).replace(",", " ")
    
    # Create data
    data = go.Bar(y=df['Pages'],
                  x=df['Pageview'],
                  text=df['Pageview'],
#                   marker=dict(color="black"),
                  orientation="h")
    # Create layout
    layout = go.Layout(
        yaxis={'categoryorder': 'total ascending'},
        margin={"l":150, "pad": 20},
        title=f"<b>Most visited web pages, by total visits</b><br><span style='font-size: 13px;'>Total visits: {value}</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        xaxis_title="No of views",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        margin_pad=10,
    )
    fig = go.Figure(data=data, layout=layout)
    fig.update_traces(textposition="outside")
    return fig

fig = plot_pageview(df_pageview)
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
