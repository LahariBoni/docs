<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/NASA/NASA_Artic_sea_ice.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=NASA+-+Artic+sea+ice:+Error+short+description">🚨 Bug report</a>

**Tags:** #nasa #naas #opendata #analytics #asset #html #png #operations #image #plotly

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

AVERAGE SEPTEMBER MINIMUM EXTENT<br>
Data source: Satellite observations. Credit: NSIDC/NASA

**What is Arctic sea ice extent?**<br>
Sea ice extent is a measure of the surface area of the ocean covered by sea ice. Increases in air and ocean temperatures decrease sea ice extent; in turn, the resulting darker ocean surface absorbs more solar radiation and increases Arctic warming.<br>
Date Range: 1979 - 2020.

## Get data from website
Click on [Artic Sea Ice](https://climate.nasa.gov/vital-signs/arctic-sea-ice/)

## Input

### Import libraries


```python
import pandas as pd
import plotly.graph_objects as go
import naas
```

### Variables


```python
# Input
data_input = "https://climate.nasa.gov/system/internal_resources/details/original/2485_Sept_Arctic_extent_1979-2021.xlsx"

# Output
image_output = "Artic_sea_ice_extent.png"
html_output = "Artic_sea_ice_extent.png"
```

## Model

### Get data


```python
df = pd.read_excel(data_input)
df.tail(10)
```

### Create graph


```python
def create_linechart(df, label, value):
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value],
            hoverinfo="text",
            hovertext=df[label].astype(str) + "<br>" + df[value].astype(str) + " million square km",
            mode="lines+text",
        )
    )
    fig.update_layout(
        title=f"<b>Arctic Sea Ice Extent</b><br><span style='font-size: 13px;'>ANNUAL SEPTEMBER MINIMUM EXTENT</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title="Year",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        yaxis_title='million square km',
        yaxis_title_font=dict(family="Arial", size=11, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df, "year", "extent")
```

## Output

### Save and share output


```python
fig.write_html(html_output)
fig.write_image(image_output)
```


```python
naas.asset.add(html_output, params={"inline": True})
naas.asset.add(image_output)

#-> Uncomment lines below to remove your assets
# naas.asset.delete(html_output)
# naas.asset.delete(image_output)
```
