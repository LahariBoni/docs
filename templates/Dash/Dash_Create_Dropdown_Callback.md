<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_Create_Dropdown_Callback.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+Create+Dropdown+Callback:+Error+short+description">🚨 Bug report</a>

**Tags:** #dashboard #plotly #dash #naas #asset #analytics #dropdown #callback #bootstrap

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

Create a basic dropdown, provide options and a value to dcc.Dropdown in that order.

<u>Reference:</u> https://dash.plotly.com/dash-core-components/dropdown

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
    import dash_bootstrap_components as dbcs
from dash import html, Input, Output, State, dcc
```

### Setup Variables


```python
DASH_PORT = 8050
```

## Model

### Create dropdown


```python
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    meta_tags=[{'name':'viewport', 'content':'width=device-width, initial-scale=1.0'}]
) 

app.layout = html.Div(
    [
        dcc.Dropdown(['NYC', 'MTL', 'SF'], 'NYC', id='demo-dropdown'),
        html.Div(id='dd-output-container')
    ]
)

@app.callback(
    Output('dd-output-container', 'children'),
    Input('demo-dropdown', 'value')
)
def update_output(value):
    return f'You have selected {value}'
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```
