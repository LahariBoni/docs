<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Metrics%20Store/Content_creation_Track_connections.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Metrics+Store+-+Content+creation+Track+connections:+Error+short+description">🚨 Bug report</a>

**Tags:** #metricsstore #metrics #content-creation #connections #content #snippet #plotly

**Author:** [Riddhi Deshpande](https://www.linkedin.com/in/riddhideshpande/)

## Input

### Import library


```python
from naas_drivers import notion
import naas
import pandas as pd
import plotly.express as px
```

### Variables


```python
# Input
notion_token = "secret_ALstzXsSXoF9zbcUakMYE1OufXHVOkb35v1rYBTzz54"
notion_database = "https://www.notion.so/naas-official/37a23cbfaac5445690301dfd49f035d3?v=8ab1b2f3847d4067ac5ae19a799c7dcb"

# Output
output_html = "ContentCreator_No_connections.csv"
```

### Get data


```python
# Get database object
db_notion = notion.connect(notion_token).database.get(notion_database)

# Get database as dataframe
df_notion = db_notion.df()
df_notion.VALUE=df_notion.VALUE.astype(float)
df_notion
```

## Model

### Create a line chart and add color parameters to distinct groups


```python
fig = px.line(data_frame=df_notion,
              x='DATE',
              y='VALUE',
              color='GROUP')

#add button control to chart
fig.update_layout(
    title=f"🚀<b> Number of Connections</b><br><span style='font-size: 13px;'></span>",
    title_font=dict(family="Arial", size=18, color="black"),
    legend_title=None,
    plot_bgcolor="#ffffff",
    width=1200,
    height=800,
    paper_bgcolor="white",
    xaxis_title="Date",
    xaxis_title_font=dict(family="Arial", size=11, color="black"),
    yaxis_title='No. of connections',
    yaxis_title_font=dict(family="Arial", size=11, color="black"),
    margin_pad=10,
    updatemenus=[
        dict(
            active=0,
            buttons=list([
                dict(
                    
                    label="Both",
                    method="update",
                    args=[{"visible":[True,True]},
                         {"title": "Both"}]),
                dict(
                    
                    label="Twitter",
                    method="update",
                    args=[{"visible":[True,False]},
                          {"title": "Twitter"}]),
                dict(
                   
                    label="Linkedin",
                    method="update",
                     args=[{"visible":[False,True]},
                          {"title": "Linkedin"}])
            ]),
            pad={"r": -80, "t": -40},
            direction="down",
            x=1,
            xanchor="left",
            y=1,
            yanchor="top",
            borderwidth=1,
            bordercolor="black",
            bgcolor=None
        ),
    ])
```

## Output

### Save and export html


```python
fig.write_html(output_html)
naas.asset.add(output_html, params={"inline": True})
```
