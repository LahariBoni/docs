<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Plotly/Plotly_Create_Bubblechart.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Plotly+-+Create+Bubblechart:+Error+short+description">🚨 Bug report</a>

**Tags:** #plotly #chart #bubblechart #dataviz #snippet #operations #image #html

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import libraries


```python
import naas
import plotly.express as px
import pandas as pd
```

### Variables


```python
title = "Life Expectancy vs GDP per Capita GDP, 2007"

# Output paths
output_image = f"{title}.png"
output_html = f"{title}.html"
```

### Get data


```python
df = px.data.gapminder()
df
```

## Model

### Create Bubblechart


```python
fig = px.scatter(
    df.query("year==2007"),
    x="gdpPercap", 
    y="lifeExp",
    size="pop",
    color="continent",
    hover_name="country",
    log_x=True,
    size_max=60
)

fig.update_layout(
    plot_bgcolor="#ffffff",
    margin=dict(l=0, r=0, t=50, b=50),
    width=1200,
    height=800,
    showlegend=False,
    xaxis_nticks=36,
    title= title,
    xaxis=dict(
        title='GDP per capita (dollars)',
        gridcolor='white',
        type='log',
        gridwidth=2,
    ),
    yaxis=dict(
        title='Life Expectancy (years)',
        gridcolor='white',
        gridwidth=2,
    ))

config = {'displayModeBar': False}
fig.show(config=config)
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
