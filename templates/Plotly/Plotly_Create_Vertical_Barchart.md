<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Plotly/Plotly_Create_Vertical_Barchart.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Plotly+-+Create+Vertical+Barchart:+Error+short+description">🚨 Bug report</a>

**Tags:** #plotly #chart #verticalbarchart #group #dataviz #snippet #operations #image #html

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
import naas
import plotly.graph_objects as go
import pandas as pd
```

### Variables


```python
title = "Sales Evolution"

# Output paths
output_image = f"{title}.png"
output_html = f"{title}.html"
```

### Get data


```python
x_axis = [1995, 1996, 1997, 1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012]
y_axis = [219, 146, 112, 127, 124, 180, 236, 207, 236, 263, 350, 430, 474, 526, 488, 537, 500, 439]
value_dict = zip(x_axis, y_axis)

df = pd.DataFrame(value_dict, columns=["LABEL", "VALUE"])
df
```

## Model

### Create Vertical Barchart


```python
def create_barchart(df, label, value, title):
    fig = go.Figure()

    fig.add_trace(go.Bar(x=df[label],
                         y= df[value],
                         marker_color='#5ee290'))
    fig.update_layout(
        title=title,
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        xaxis_tickfont_size=14,
        legend=None,
        bargap=0.1, # gap between bars of adjacent location coordinates.
        bargroupgap=0.1 # gap between bars of the same location coordinate.
    )
    config = {'displayModeBar': False}
    fig.show(config=config)
    return fig

fig = create_barchart(df, label="LABEL", value="VALUE", title=title)
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
