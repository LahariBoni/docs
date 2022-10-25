<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_Deploy_app_in_Naas.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+Deploy+app+in+Naas:+Error+short+description">🚨 Bug report</a>

**Tags:** #dashboard #plotly #dash #naas #asset #automation #analytics

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/jeremyravenel/)

This notebook enables you to deploy a Dash app using Naas proxy.

## Input

### Import libraries


```python
import os
try:
    import dash
except:
    !pip install dash --user
    import dash
try:
    import dash_bootstrap_components as dbc
except:
    !pip install dash_bootstrap_components --user
    import dash_bootstrap_components as dbc
from dash import html, dcc
import plotly.express as px
import plotly.graph_objects as go
```

### Setup Variables


```python
DASH_PORT = 8050
```

## Model

### Initialize Dash app


```python
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    meta_tags=[{'name':'viewport', 'content':'width=device-width, initial-scale=1.0'}]
) 

#app = dash.Dash() if you are not in Naas
```

### Get stock prices


```python
df = px.data.stocks() #reading stock price dataset
print("Data fetched:", len(df))
df.head(1)
```

### Create chart


```python
def stock_prices(ticker, label):
    # Function for creating line chart showing Google stock prices over time 
    fig = go.Figure(
        [
            go.Scatter(
                x=df['date'],
                y=df[ticker],
                line=dict(color='firebrick', width=4),
                name=label
            )
        ]
    )
    fig.update_layout(
        title='Prices over time',
        xaxis_title='Dates',
        yaxis_title='Prices'
    )
    return fig

fig = stock_prices(ticker='GOOG', label="Google")
fig
```

### Create app layout


```python
app.layout = html.Div(
    id ='parent',
    children=[
        html.H1(
            id='H1',
            children='Deploy a Dash app in Naas',
            style = {
                'textAlign': 'center',
                'marginTop': 40,
                'marginBottom': 40
            }
        ),
        dcc.Graph(
            id='line_plot', 
            figure=fig
        )    
    ]
)
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```


```python

```
