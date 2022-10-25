<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_Plotly_Dynamic_Link.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+Plotly+Dynamic+Link:+Error+short+description">🚨 Bug report</a>

**Tags:** #dash #plotly #naas #analytics

**Author:** [Oguz Akif Tufekcioglu](https://www.linkedin.com/in/oguzakiftufekcioglu/)

This notebook enables you to generate plotly plots with clickable data points that sends to their urls in the dataframe.

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
    
import webbrowser
from dash.dependencies import Input, Output
from dash import html, dcc
from dash.exceptions import PreventUpdate
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

# if you are not in Naas
# app = dash.Dash() 
```

### Get stock prices


```python
df = px.data.stocks() #reading stock price dataset
print("Data fetched:", len(df))
df.head()
```

### Add an url column


```python
# add a url for each stock in the dataset into urls column
df["urls"] = "https://www.naas.ai/"
df.head()
```

### Create app layout


```python
app.layout = html.Div(
    id ='parent',
    children=[
        html.H1(
            id='H1',
            children='Dynamic link on label click on plotly chart python',
            style = {
                'textAlign': 'center',
                'marginTop': 40,
                'marginBottom': 40
            }
        ),
        dcc.Graph(
            id='line-plot', 
            figure=px.line(
                data_frame=df,
                x='date',
                y='GOOG',
                title='Google stock prices over time',
                hover_name="urls",
                custom_data=("urls",)
            )
        ),
        dcc.Store(
        id='clientside-data',
        data=""
        ),
        dcc.Store(
        id='redirected-url',
        data=""
        ),
    ]
)
```

### Call callback function to save url in a dcc Store component when data point is clicked


```python
@app.callback(Output('clientside-data', 'data'), [Input('line-plot', 'clickData')])
def open_url(clickData):
    if clickData != None:
        print(clickData)
        url = clickData['points'][0]['customdata'][0]
        return url
    else:
        raise PreventUpdate
```

### Call client-side callback function open url in a new tab


```python
app.clientside_callback(
    """
    function(data,scale) {
      console.log(data);
      window.open(data, '_blank');
      return data;
    }
    """,
    Output("redirected-url", 'data'),
    Input("clientside-data", 'data'),
)
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```
