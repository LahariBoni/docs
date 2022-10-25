<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Plotly/Plotly_Create_Barline_chart.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Plotly+-+Create+Barline+chart:+Error+short+description">🚨 Bug report</a>

**Tags:** #plotly #naas #snippet #operations #barline

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel)

As a business user, I want to see a graph with 2 y-axis. This can show for example the evolution and variation of sales.

- one is a line with a trend (1000 → 100000)
- one is a bar with variation (100 → x)

The notebook is divided in 3 parts: 

- Input : setup data frame with one column with DATE, VALUE, VAR
- Model: create barline chart from plotly library
- Output: generate PNG and HTML files via URL link.

## Input

### Import libraries


```python
import pandas as pd
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import naas
```

### Create dataframe


```python
data = [
    {"DATE": "202201", "VALUE": 130, "VARV": 130},
    {"DATE": "202202", "VALUE": 180, "VARV": 50},
    {"DATE": "202203", "VALUE": 190, "VARV": 10},
    {"DATE": "202204", "VALUE": 200, "VARV": 10},
    {"DATE": "202205", "VALUE": 280, "VARV": 80},
]
df = pd.DataFrame(data)
df
```

### Setup Outputs


```python
html_output = "Barline.html"
image_output = "Barline.png"
```

## Model

### Create barlinechart using Plotly


```python
def create_barlinechart(df,
                        label="DATE",
                        value="VALUE",
                        varv="VARV",
                        xaxis_title="Months",
                        yaxis_title_r=None,
                        yaxis_title_l=None):    
    # Create figure with secondary y-axis
    fig = make_subplots(specs=[[{"secondary_y": True}]])

    # Add traces
    fig.add_trace(
        go.Bar(
            x=df[label],
            y=df[varv],
            marker=dict(color="#ADD8E6"),
        ),
        secondary_y=False,
    )
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value],
            mode="lines",
            line=dict(color="#0A66C2", width=2.5),
        ),
        secondary_y=True,
    )

    # Add figure title
    fig.update_layout(
        title="Plotly - Barline chart",
        title_font=dict(family="Arial", size=18, color="black"),
        legend=None,
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title=xaxis_title,
        xaxis_title_font=dict(family="Arial", size=10, color="black"),
    )

    # Set y-axes titles
    fig.update_yaxes(
        title_text=yaxis_title_r,
        title_font=dict(family="Arial", size=10, color="black"),
        secondary_y=False
    )
    fig.update_yaxes(
        title_text=yaxis_title_l,
        title_font=dict(family="Arial", size=10, color="black"),
        secondary_y=True
    )
    fig.update_traces(showlegend=False)
    fig.show()
    return fig

fig = create_barlinechart(df,
                          yaxis_title_r="Variation",
                          yaxis_title_l="Value")
```

## Output

### Save and share your graph in HTML


```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```

### Save and share your graph in image


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
naas.asset.add(image_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```


```python

```
