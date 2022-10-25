<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_LinkedIn_posts_metrics_dashboard.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+LinkedIn+posts+metrics+dashboard:+Error+short+description">🚨 Bug report</a>

**Tags:** #dash #linkedin #dashboard #plotly #naas #asset #automation #analytics

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook enables you to generate a dashboard to follow your LinkedIn Posts Metrics with 4 cards and 4 charts.

## Input

### Import libraries


```python
# Data
import os
from os import environ
import pandas as pd
import naas
from datetime import datetime
from naas_drivers import gsheet

# Plotly
import plotly.graph_objects as go
import plotly.express as px
from plotly.subplots import make_subplots

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
from dash import html, dcc, Input, Output, State
from dash_bootstrap_components._components.Container import Container
from dash.exceptions import PreventUpdate
```

### Setup Dash App
Elements displayed in Dash App


```python
# Defining the port of the dashboard
APP_NAME = 'LinkedIn'
APP_TITLE = 'Posts Metrics Analysis'
APP_LOGO = "https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/LinkedIn_logo_initials.png/800px-LinkedIn_logo_initials.png"
COLOR = "#1293d2"
COLOR2 = "#cfe9f6"
BACKGROUND_COLOR = "#f8f9fa"

# Navbar
NAVBAR_COLOR = "white"
NAVBAR_TOGGLE_BD_COLOR = "#f8f9fa"

# Card
CARD_COLOR = "white"

# Chart
CHART_COLOR = "white" 

# Port used to displayed preview
DASH_PORT = 8050 
```

### Setup Google Sheets


```python
SPREADSHEET_URL = "https://docs.google.com/spreadsheets/d/1yR29Kfg8iHPiMsl5n_xIsdWIjxKgMbMphRr7_q-4SBs/edit#gid=1582298725"
SH_ENTITIES = 'ENTITIES'
SH_SCENARIOS = 'SCENARIOS'
SH_POSTS_PUBLISHED = "POSTS_PUBLISHED"
SH_POSTS_VIEWS = "POSTS_VIEWS"
SH_POSTS_LIKES = "POSTS_LIKES"
SH_POSTS_COMMENTS = "POSTS_COMMENTS"
```

## Model

### Constants


```python
DATE_FORMAT = "%Y-%m-%d"
PERIOD = "%Y%m"
PERIOD_D = "%b %Y"
PERIOD_TEXT = "This Month"
PERIOD_COMP = "last month"
```

### Data

#### Ref Data: Entities


```python
df_entities = gsheet.connect(SPREADSHEET_URL).get(SH_ENTITIES)
entities = sorted(df_entities["ENTITY"].unique(), reverse=True)
entities
```

#### Ref Data: Scenarios


```python
df_scenarios = gsheet.connect(SPREADSHEET_URL).get(SH_SCENARIOS)
scenarios = df_scenarios["SCENARIO"].unique().tolist()
scenarios
```

#### Posts published


```python
df_1 = gsheet.connect(SPREADSHEET_URL).get(SH_POSTS_PUBLISHED)
print("Rows fetched:", len(df_1))
df_1.head(1)
```

#### Posts views


```python
df_2 = gsheet.connect(SPREADSHEET_URL).get(SH_POSTS_VIEWS)
print("Rows fetched:", len(df_2))
df_2.head(1)
```

#### Posts likes


```python
df_3 = gsheet.connect(SPREADSHEET_URL).get(SH_POSTS_LIKES)
print("Rows fetched:", len(df_3))
df_3.head(1)
```

#### Posts comments


```python
df_4 = gsheet.connect(SPREADSHEET_URL).get(SH_POSTS_COMMENTS)
print("Rows fetched:", len(df_4))
df_4.head(1)
```

### Charts

#### Create barchart


```python
def create_barchart(df,
                    title=None,
                    label="DATE_D",
                    value="VALUE",
                    value_d="VALUE_D"):
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Bar(
            x=df[label],
            y=df[value],
            text=df[value_d],
            textposition="outside",
            marker=dict(color=COLOR),
        )
    )
    fig.update_traces(showlegend=False)
    
    # Update layout
    fig.update_layout(
        title=title,
        title_font=dict(family="Arial", color="black"),
        paper_bgcolor=CHART_COLOR,
        plot_bgcolor=CHART_COLOR,
        xaxis=dict(showgrid=False),
        yaxis=dict(showgrid=False),
        margin=dict(l=20, r=20, t=20, b=20)
    )
    fig.update_yaxes(showticklabels=False)
    config = {'displayModeBar': False, "responsive": True}
    fig.show(config=config)
    return fig

# Create chart (test)
df_barchart = df_1[(df_1.ENTITY == entities[0]) & (df_1.SCENARIO == scenarios[0])]
fig = create_barchart(df_barchart)
```

### Dash App

#### Create dropdown "Entity"


```python
dropdown_entity = dcc.Dropdown(
    id='entity',
    options=[{'label': i, 'value': i} for i in entities],
    placeholder='Entity',
    value=entities[0],
    style={
        "text-align": "center",
        "background-color": "#EEF3F8",
        "color": "light"
    }
)
```

#### Create dropdown "Scenario"


```python
dropdown_scenario = dcc.Dropdown(
    id='scenario',
    options=[{'label': i, 'value': i} for i in scenarios],
    placeholder='Scenario',
    value=scenarios[0],
    style={
        "text-align": "center",
        "background-color": "#EEF3F8",
        "color": "light"
    }
)
```

#### Create Navbar


```python
navbar = dbc.Navbar(
    dbc.Container(
        [
            html.A(
                # Use row and col to control vertical alignment of logo / brand
                dbc.Row(
                    [
                        dbc.Col(
                            html.Img(
                                src=APP_LOGO,
                                height="30px"
                            )
                        ),
                        dbc.Col(
                            dbc.NavbarBrand(
                                "LinkedIn",
                                className="ms-2",
                                style={
                                    "color": COLOR,
                                    "font-weight": "bold"
                                }
                            )
                        ),
                    ],
                    align="center",
                    className="g-0",
                ),
                href="https://github.com/jupyter-naas/awesome-notebooks/tree/master/Dash",
                style={"textDecoration": "none"},
            ),
            dbc.NavbarToggler(
                id="navbar-toggler",
                n_clicks=0,
                style={
                    "border-color": NAVBAR_TOGGLE_BD_COLOR,
                    "background": COLOR2,
                    "margin-right": "10px"
                }
            ),
            dbc.Collapse(
                dbc.Nav(
                    [
                        html.Div(
                            [
                                html.Div(className="w-100"),
                                html.Div(className="w-100"),
                                html.Div(dropdown_entity, className="w-100"),
                                html.Div(dropdown_scenario, className="w-100")
                            ],
                            className="pt-1 pb-1 d-grid gap-2 d-md-flex w-100")

                    ],
                    className="ms-auto w-100",
                    navbar=True,
                ),
                id="navbar-collapse",
                navbar=True,
                is_open=False,
            ),
        ],
    ),
    color=NAVBAR_COLOR,
    dark=True,
    className="navbar border-bottom border-light fixed-top",
    style={}
)
```

#### Create App title


```python
def create_app_title(name, uid="app_title"):
    title = dbc.Row(
            html.H2(
                name,
                id=uid,
                style={
                    "color": COLOR,
                    "background-color": BACKGROUND_COLOR,
                    "font-weight": "bold",
                }
            ),
            className="text-center",
            style={
                "background-color": BACKGROUND_COLOR,
                "padding-top": "70px",
                "padding-bottom": "10px"
            },
        )
    return title
```

#### Create Section


```python
def create_section(name, uid=None):
    section = dbc.Row(
        [
            dbc.Col(
                html.Hr(),
                xs=4, sm=4, md=5, lg=5, xl=5
            ),
            dbc.Col(
                html.H4(
                    name,
                    id=uid,
                    className="text-center",
                    style={
                        "color": COLOR,
                        "background-color": BACKGROUND_COLOR,
                        "margin-bottom": "0px"
                    }
                ), 
                xs=4, sm=4, md=2, lg=2, xl=2
            ),
            dbc.Col(
                html.Hr(),
                xs=4, sm=4, md=5, lg=5, xl=5
            )
        ],
        style={
            "background-color": BACKGROUND_COLOR,
            "padding-top": "8px",
            "padding-bottom": "8px"
        },
        className="align-items-center",
    )
    return section
```

#### Create HKPIs


```python
def create_hkpis(card_title, card_value, card_var):
    card = dbc.Col(
        dbc.Card(
            dbc.CardBody(
                [
                    html.P(
                        card_title,
                        className="card-title",
                        style={
                            "font-size": "15px",
                            "font-weight": "bold",
                            "padding-bottom": "5px",
                           }
                    ),
                    html.P(
                        className="card-text",
                        style={},
                        id=card_var
                    ),
                    html.P(
                        className="card-text",
                        style={
                            "font-size": "14px",
                           },
                        id=card_value
                    )
                ]
            ),
            color=CARD_COLOR,
            inverse=False
        ),
        style={
            "background-color": BACKGROUND_COLOR
        },
        className="gx-5 g-2",
        xs=12,
        sm=12,
        md=12,
        lg=3,
        xl=3
    )
    return card
```

#### Create Chart


```python
def create_chart(title, chart_id):
    card = dbc.Col(
        dbc.Card(
            dbc.CardBody(
                [
                    html.H5(
                        title,
                        style={
                            "padding-top": "20px",
                            "margin-left": "20px",
                            "font-weight": "bold",
                        }
                    ),
                    dcc.Graph(
                        id=chart_id,
                        config={
                            'displayModeBar': False,
                            "staticPlot": True
                        }
                    ),
                ],
            ),
            color=CARD_COLOR,
            inverse=False
        ),
        style={
            "background-color": BACKGROUND_COLOR
        },
        className="gx-5 g-2",
        xs=12,
        sm=12,
        md=12,
        lg=6,
        xl=6
    )
    return card
```

#### Create Layout


```python
# Init App
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    assets_external_path="chart.css",
    meta_tags=[{'name': 'viewport','content': 'width=device-width, initial-scale=1.0'}]
                
)

# App title
app.title = APP_TITLE

# App layout
app.layout = html.Div(
    [
        # Navbar
        navbar,
        
        # App title
        create_app_title(APP_TITLE),
        
        # Section name
        create_section("Overview", "section1"),

        # HKPIS
        dbc.Row(
            [
                create_hkpis("PUBLISHED", "card_val1", "card_var1"),
                create_hkpis("VIEWS", "card_val2", "card_var2"),
                create_hkpis("LIKES", "card_val3", "card_var3"),
                create_hkpis("COMMENTS", "card_val4", "card_var4"),
            ],
            style={
                "background-color": BACKGROUND_COLOR,
                "padding-top": "8px",
                "padding-bottom": "8px"
            },
            className="align-items-center",
        ),
        
        # Section name
        create_section("Charts", "section2"),
        
        # Charts 
        dbc.Row(
            [
                create_chart("POST PUBLISHED", "fig1"),
                create_chart("POST VIEWS", "fig2"),

            ],
            style={
                "background-color": BACKGROUND_COLOR,
                "padding-top": "8px",
                "padding-bottom": "8px"
            },
        ),
        dbc.Row(
            [
                create_chart("POST LIKES", "fig3"),
                create_chart("POST COMMENTS", "fig4"),
            ],
            style={
                "background-color": BACKGROUND_COLOR,
                "padding-top": "8px",
                "padding-bottom": "8px"
            },
        )
    ]
)

# add callback for toggling the collapse on small screens
@app.callback(
    Output("navbar-collapse", "is_open"),
    [Input("navbar-toggler", "n_clicks")],
    [State("navbar-collapse", "is_open")],
)
def toggle_navbar_collapse(n, is_open):
    if n:
        return not is_open
    return is_open

@app.callback(
    [
        Output('card_val1', 'children'),
        Output('card_var1', 'children'),
        Output('card_val2', 'children'),
        Output('card_var2', 'children'),
        Output('card_val3', 'children'),
        Output('card_var3', 'children'),
        Output('card_val4', 'children'),
        Output('card_var4', 'children'), 
        Output('fig1', 'figure'),
        Output('fig2', 'figure'), 
        Output('fig3', 'figure'),
        Output('fig4', 'figure'),
    ],
    [
        Input('entity', 'value'),
        Input('scenario', 'value')
    ]
    
)

def multi_outputs(entity, scenario):
    if entity is None and scenario is None:
        raise PreventUpdate

    # Get graph dataframe
    def filter_dropdowns(df, entity, scenario):
        df = df[(df["ENTITY"] == entity) & (df["SCENARIO"] == scenario)].reset_index(drop=True)
        return df
    
    df1 = filter_dropdowns(df_1, entity, scenario)
    df2 = filter_dropdowns(df_2, entity, scenario)
    df3 = filter_dropdowns(df_3, entity, scenario)
    df4 = filter_dropdowns(df_4, entity, scenario)
    
    # Return card data
    card_val1 = df1.loc[df1.index[-1], "HKPI_VALUE"]
    card_var1 = df1.loc[df1.index[-1], "HKPI_VAR"]
    card_val2 = df2.loc[df2.index[-1], "HKPI_VALUE"]
    card_var2 = df2.loc[df2.index[-1], "HKPI_VAR"]
    card_val3 = df3.loc[df3.index[-1], "HKPI_VALUE"]
    card_var3 = df3.loc[df3.index[-1], "HKPI_VAR"]
    card_val4 = df4.loc[df4.index[-1], "HKPI_VALUE"]
    card_var4 = df4.loc[df4.index[-1], "HKPI_VAR"]

    # Create graph
    fig1 = create_barchart(df1)
    fig2 = create_barchart(df2)
    fig3 = create_barchart(df3)
    fig4 = create_barchart(df4)
    return card_val1, card_var1, card_val2, card_var2, card_val3, card_var3, card_val4, card_var4, fig1, fig2, fig3, fig4
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```
