<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Plotly/Plotly_Create_Heatmap.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Plotly+-+Create+Heatmap:+Error+short+description">🚨 Bug report</a>

**Tags:** #plotly #chart #heatmap #dataviz #snippet #operations #image #html

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
import naas
import plotly.graph_objects as go
import plotly.express as px
import pandas as pd
```

### Variables


```python
title = "Heatmap"

# Output paths
output_image = f"{title}.png"
output_html = f"{title}.html"
```

### Get data


```python
data = [[54.25, 65, 52, 51, 49],
        [20, 16, 60, 80, 30],
        [30, 60, 51, 59, 20],
        [40, 30, 12, 25, 20]]
df = pd.DataFrame(data)
df
```

## Model

### Create Heatmap


```python
config = {'displayModeBar': False}

def update_layout(fig, title=None, plot_bgcolor=None, width=1200, height=800, showlegend=None, margin=None):
    fig.update_layout(
        title=title,
        plot_bgcolor=plot_bgcolor,
        width=width,
        margin=margin,
        height=height,
        showlegend=showlegend)
    return fig

def heatmap(df,
            x,
            y,
            x_label,
            y_label,
            color,
            colorscale=[[0, "#d94228"],[0.5, "#d94228"],[0.5, "#5ee290"],[1.0, "#5ee290"]],
            title=None,
            margin=None):
    fig = go.Figure()
    fig = px.imshow(df,
                    labels=dict(x=x_label, y=y_label, color=color),
                    x=x,
                    y=y)
    fig.update_xaxes(side="top")
    fig.update_traces(xgap=5, selector=dict(type='heatmap'))
    fig.update_traces(ygap=5, selector=dict(type='heatmap'))
    fig.update_traces(dict(showscale=False, 
                           coloraxis=None, 
                           colorscale=colorscale),
                           selector={'type':'heatmap'})
    fig = update_layout(fig, title=title, margin=margin)
    fig.show(config=config)
    return fig

fig = heatmap(df,
              x=['Global', 'Americas', 'EMEA', 'Asia', 'Oceania'],
              y=['Product 1', 'Product 2', 'Product 3', 'Product 4'],
              x_label="Business lines",
              y_label="Products",
              color="Sales",
              title=title,
              margin=dict(l=0, r=0, t=150, b=50))   
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
