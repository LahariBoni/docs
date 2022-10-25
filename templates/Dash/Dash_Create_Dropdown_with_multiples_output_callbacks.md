<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_Create_Dropdown_with_multiples_output_callbacks.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+Create+Dropdown+with+multiples+output+callbacks:+Error+short+description">🚨 Bug report</a>

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
    import dash_bootstrap_components as dbc
from dash import html, Input, Output, State, dcc, dash_table as dt
import plotly.graph_objs as go
from dash.exceptions import PreventUpdate
```

### Setup Variables


```python
DASH_PORT = 8050
```

## Model

### Data


```python
sample_data = {
    'series': {
        'data': [
            {'title': 'Game of Thrones', 'score': 9.5},
            {'title': 'Stranger Things', 'score': 8.9},
            {'title': 'Vikings', 'score': 8.6}
        ],
        'style': {
            'backgroundColor': '#ff998a'
        }
    },
    'movies': {
        'data': [
            {'title': 'Rambo', 'score': 7.7},
            {'title': 'The Terminator', 'score': 8.0},
            {'title': 'Alien', 'score': 8.5}
        ],
        'style': {
            'backgroundColor': '#fff289'
        }
    }
}
```

### Create dropdown


```python
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    meta_tags=[{'name':'viewport', 'content':'width=device-width, initial-scale=1.0'}]
) 

app.layout = html.Div(
    [
        html.H1('Multi output example'),
        dcc.Dropdown(
            id='data-dropdown',
            options=[
                 {'label': 'Movies', 'value': 'movies'},
                 {'label': 'Series', 'value': 'series'}
            ],
            value='movies'
        ),
        html.Div(
            [
                dcc.Graph(id='graph'),
                dt.DataTable(
                    id='data-table',
                    columns=[
                        {'name': 'Title', 'id': 'title'},
                        {'name': 'Score', 'id': 'score'}
                    ]
                )
            ]
        )
    ],
    id='container'
)

@app.callback(
    [
        Output('graph', 'figure'),
        Output('data-table', 'data'),
        Output('data-table', 'columns'),
        Output('container', 'style')
    ],
    [
        Input('data-dropdown', 'value')
    ]
)

def multi_outputs(value):
    if value is None:
        raise PreventUpdate
    
    # Display table data
    selected = sample_data[value]
    data = selected['data']
    columns = [{'name': k.capitalize(), 'id': k} for k in data[0].keys()]
    
    # Display figure
    figure = go.Figure(
        data=[
            go.Bar(x=[x['score']], text=x['title'], name=x['title'])
            for x in data
        ]
    )
    return figure, data, columns, selected['style']
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```


```python

```
