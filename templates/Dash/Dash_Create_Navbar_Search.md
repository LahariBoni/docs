<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_Create_Navbar_Search.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+Create+Navbar+Search:+Error+short+description">🚨 Bug report</a>

**Tags:** #dash #snippet #dashboard #plotly #dash #analytics

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

Create your Navbar search like Google using Dash App.

## Input

### Import libraries


```python
# Dash
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
from dash.dependencies import Input, Output

# Other
import os
```

### Setup Dash


```python
# App variables
DASH_PORT = 8050
APP_TITLE = "Naas Search Content"

# Search variables
placeholder = "  🔍  Find something..."
naas_logo_url = "https://github.com/jupyter-naas/naas-search-insights/blob/9ade37588d13698a4308b160978b4ad34176ecc3/inputs/naas-bg-white.png?raw=true"
```

## Model

### Create Input Search


```python
input_search = dcc.Input(
                    id="result",
                    type="search",
                    className="form-control rounded-pill",
                    placeholder=placeholder,
                    style={
                        'marginRight':'10px',
                        "padding-left": "20px"
                    }
                )
```

#### Create Navbar


```python
navbar = dbc.Navbar(
    dbc.Row(
        [
            dbc.Col(
                html.Img(
                    src=naas_logo_url,
                    height="30px",
                    style={
                        'marginLeft': 'auto',
                        'marginRight': 'auto',
                        'display': 'block',
                    }
                ),
                xs=12,
                sm=12,
                md=2,
                lg=2,
                xl=2
            ),
            dbc.Col(
                input_search,
                xs=12,
                sm=12,
                md=10,
                lg=10,
                xl=8
            ),
            dbc.Col(
                xs=0,
                sm=0,
                md=1,
                lg=1,
                xl=2,
            ),
        ],
        style={
            "display": "contents"
        },
        align="center",
        className="g-3",
    ),
    color="white",
    dark=True,
    className="navbar border-bottom border-light",

)
```

#### Create App layout


```python
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    meta_tags=[{'name':'viewport', 'content':'width=device-width, initial-scale=1.0'}]
) 
app.title = APP_TITLE
app.layout = html.Div(
    [
        navbar,
    ]
)
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```
