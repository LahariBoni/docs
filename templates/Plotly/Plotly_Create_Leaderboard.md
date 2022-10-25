<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Plotly/Plotly_Create_Leaderboard.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Plotly+-+Create+Leaderboard:+Error+short+description">🚨 Bug report</a>

**Tags:** #plotly #chart #horizontalbar #dataviz #snippet #operations #image #html

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

Learn more on the Plotly doc : https://plotly.com/python/horizontal-bar-charts/

## Input

### Import libraries


```python
import plotly.express as px
import pandas as pd
```

### Variables


```python
title = "Leaderboard"

# Output paths
output_image = f"{title}.png"
output_html = f"{title}.html"
```

## Model

### Get data model


```python
data = [
    {"LABEL": "A", "VALUE": 88},
    {"LABEL": "B", "VALUE": 12},
    {"LABEL": "C", "VALUE": 43},
    {"LABEL": "D", "VALUE": 43},
    {"LABEL": "E", "VALUE": 2},
    {"LABEL": "F", "VALUE": 87},
    {"LABEL": "G", "VALUE": 67},
    {"LABEL": "H", "VALUE": 111},
    {"LABEL": "I", "VALUE": 24},
    {"LABEL": "J", "VALUE": 123},
]
df = pd.DataFrame(data)
df = df.sort_values(by=["VALUE"], ascending=True) #Order will be reversed in plot 
df
```

### Create the plot


```python
def create_barchart(df, label, value):
    last_value = '{:,.0f}'.format(df[value].sum())
    fig = px.bar(df,
                 y=label,
                 x=value,
                 orientation='h',
                 text=value)
    fig.update_layout(
        title=f"<b>Ranking by label</b><br><span style='font-size: 13px;'>Total value: {last_value}</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        legend_title="Packs",
        legend_title_font=dict(family="Arial", size=11, color="black"),
        legend_font=dict(family="Arial", size=10, color="black"),
        font=dict(family="Arial", size=12, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        xaxis_title=None,
        xaxis_showticklabels=False,
        yaxis_title=None,
        margin_pad=10,
        margin_t=100,
    )
    # Display fig        
    config = {'displayModeBar': False}
    fig.show(config=config)
    return fig

fig = create_barchart(df, "LABEL", "VALUE")
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
