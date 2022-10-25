<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Plotly/Plotly_Create_Linechart.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Plotly+-+Create+Linechart:+Error+short+description">🚨 Bug report</a>

**Tags:** #plotly #chart #linechart #trend #dataviz #yahoofinance #naas_drivers #snippet #operations #image #html

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

Learn more on the Plotly doc : https://plotly.com/python/line-charts/

## Input

### Import libraries


```python
import naas
from naas_drivers import yahoofinance, plotly
```

### Variables


```python
title = "Linechart"

# Output paths
output_image = f"{title}.png"
output_html = f"{title}.html"
```

### Get data


```python
date_from = -360 # Date can be number or date or today
date_to = "today"
df = yahoofinance.get("TSLA",
                      date_from=date_from,
                      date_to=date_to)
df
```

## Model


```python
fig = plotly.linechart(df,
                       x="Date",
                       y=["Open", "Close"],
                       title=title,
                       yaxis_title="Price in $")
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
