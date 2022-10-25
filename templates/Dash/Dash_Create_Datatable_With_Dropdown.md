<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_Create_Datatable_With_Dropdown.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+Create+Datatable+With+Dropdown:+Error+short+description">🚨 Bug report</a>

**Tags:** #dash #dashboard #plotly #naas #asset #analytics #dropdown #callback #bootstrap #snippet

**Author:** [Ismail Chihab](https://www.linkedin.com/in/ismail-chihab-4b0a04202/)

Create a basic table that can be updated through a dcc.dropdown menu.

<u>Reference:</u>
- https://dash.plotly.com/datatable
- https://stackoverflow.com/questions/72013085/update-datatable-with-sub-dropdowns-dash-python

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
from dash import html, Input, Output, dcc, dash_table
import pandas as pd
```

### Setup Dash App


```python
DASH_PORT = 8050
```

## Model

### Data


```python
column_name = ["col1", "col2", "col3"]
data = [[0, 1, 2], [0, 6, 4], [6, 7, 1], [0, 1, 2], [3, 4, 5]]
df = pd.DataFrame(data, columns=column_name)
df
```

### Dash App

#### Create dropdown


```python
dropdown = dcc.Dropdown(['option1', 'option2', 'option3'], 'option1', id='demo-dropdown')
```

#### Create table


```python
table = dash_table.DataTable(id='tbl', data=df.to_dict('records') , columns=[{"name": i, "id": i} for i in df.columns])
```

#### Create Layout


```python
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    meta_tags=[{'name':'viewport', 'content':'width=device-width, initial-scale=1.0'}]
)

app.layout = html.Div(
    [
        dropdown,
        table
    ]
)

@app.callback(
    Output('tbl', 'data'),
    Input('demo-dropdown', 'value')
)

def update_output(value):
    dff = df.copy()
    if str(value) == 'option1':
        dff = dff[dff['col1'] == 0]
    elif str(value) == 'option2':
        dff = dff[dff['col2'] == 3]
    return dff.to_dict('records')
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```
