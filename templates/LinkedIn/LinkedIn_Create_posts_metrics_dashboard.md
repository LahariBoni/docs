<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Create_posts_metrics_dashboard.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Create+posts+metrics+dashboard:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #dashboard #plotly #dash #naas #asset #automation #analytics

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook enables you to generate a dashboard to follow your LinkedIn Posts Metrics.

→ Get more examples of how to build a dashboard in Dash Plotly in the Dash folder. <br>
→ To push your dashboard using Knative in naas please contact us (florent@naas.ai)

<div class="alert alert-info" role="info" style="margin: 10px">
<b>Requirements:</b><br>
To run this notebook, you must have already run <b>LinkedIn_Get_profile_posts_stats.ipynb</b> or <b>LinkedIn_Get_company_posts_stats.ipynb</b> to get your post stats in CSV.<br>
</div>

## Input

### Import libraries


```python
# Data
import os
from os import environ
import pandas as pd
import naas
from datetime import datetime

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

### Setup CSV
List of CSVs storing your posts stats. <br>
*NB: You must have already run <b>LinkedIn_Get_profile_posts_stats.ipynb</b> or <b>LinkedIn_Get_company_posts_stats.ipynb</b> to get your post stats in CSV.<br>*


```python
# Files
CSV_FILES = [
    "LINKEDIN_COMPANY_POSTS_70506391.csv",
    "LINKEDIN_PROFILE_POSTS_ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg.csv",
    "LINKEDIN_PROFILE_POSTS_ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog.csv"
]
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

#### Raw data


```python
def concat_csv(files):
    df = pd.DataFrame()
    for file in files:
        try:
            tmp_df = pd.read_csv(file)
            df = pd.concat([df, tmp_df])
        except Exception as e:
            print(e)
    return df

df_posts = concat_csv(CSV_FILES)
print("Posts fetched:", len(df_posts))
df_posts.head(1)
```

#### Master data


```python
def create_master_table(df):
    df_master = df.copy()

    # Add entity and scenario
    df_master["ENTITY"] = df_master["AUTHOR_NAME"]
    df_master["SCENARIO"] = pd.to_datetime(df_master["PUBLISHED_DATE"].str[:7], format="%Y-%m").dt.strftime("%b %Y")
    df_master["SCENARIO_ORDER"] = pd.to_datetime(df_master["PUBLISHED_DATE"].str[:7], format="%Y-%m").dt.strftime(PERIOD)
    df_master = df_master[df_master["VIEWS"] != 0]

    # Clean
    df_master = df_master.drop_duplicates("ACTIVITY_ID")
    return df_master

df_master = create_master_table(df_posts)
print("Posts fetched clean:", len(df_master))
df_master.head(1)
```

#### Ref Data: Entities


```python
entities = sorted(df_master["ENTITY"].unique(), reverse=True)
entities
```

#### Ref Data: Scenarios


```python
scenarios = df_master[["SCENARIO_ORDER", "SCENARIO"]].drop_duplicates().sort_values(by="SCENARIO_ORDER", ascending=False)["SCENARIO"].tolist()[:12]
scenarios
```

#### Barchart data


```python
def get_trend_barchart(df_init,
                       col_date,
                       col_value,
                       agg_value,
                       entity=None,
                       scenario=None,
                       period_rolling=12):
    # Init variable
    df = df_init.copy()
    
    # Filter on entity
    df = df[(df.ENTITY == entity)]
    
    # Groupby period
    if isinstance(col_value, list):
        df["VALUE"] = 0
        for c in col_value:
            df[c] = df[c].astype(float)
            df["VALUE"] = df["VALUE"] + df[c]    
        col_value = "VALUE"
    elif agg_value == "sum":
        df[col_value] = df[col_value].astype(float)
    df[col_date] = pd.to_datetime(df[col_date].str[:-6]).dt.strftime(DATE_FORMAT)
    df = df.groupby(col_date, as_index=False).agg({col_value: agg_value})
    
    # Rename column
    to_rename = {
        col_date: "DATE_ISO",
        col_value: "VALUE"
    }
    df = df.rename(columns=to_rename)
    
    # Reindex value
    d = datetime.now().date()
    d2 = df.loc[df.index[0], "DATE_ISO"]
    idx = pd.date_range(d2, d, freq = "D")    
    df.set_index("DATE_ISO", drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df["DATE_ISO"] = pd.DatetimeIndex(df.index)
    
    # Groupby month
    df["DATE"] = pd.to_datetime(df["DATE_ISO"], format=DATE_FORMAT).dt.strftime(PERIOD)
    # Plotly: Date display
    df["DATE_D"] = pd.to_datetime(df["DATE_ISO"], format=DATE_FORMAT).dt.strftime(PERIOD_D)
    df = df.groupby(["DATE", "DATE_D"], as_index=False).agg({"VALUE": "sum"})

    # Calc variation
    for index, row in df.iterrows():
        if index > 0:
            n = df.loc[df.index[index], "VALUE"]
            n_1 = df.loc[df.index[index-1], "VALUE"]
            df.loc[df.index[index], "VALUE_COMP"] = n_1
            df.loc[df.index[index], "VARV"] = n - n_1
            if n_1 > 0:
                df.loc[df.index[index], "VARP"] = (n - n_1) / abs(n_1)
    df = df.fillna(0)
    
    # Plotly: Value display
    df["VALUE_D"] = "<b><span style='font-family: Arial;'>" + df["VALUE"].map("{:,.0f}".format).str.replace(",", " ") + "</span></b>"
    
    # Plotly: Variation display
    df["VARV_D"] = df["VARV"].map("{:,.0f}".format).str.replace(",", " ")
    df.loc[df["VARV"] > 0, "VARV_D"] = "🟢 +" + df["VARV_D"]
    df.loc[df["VARV"] == 0, "VARV_D"] = "🟠 " + df["VARV_D"]
    df.loc[df["VARV"] < 0, "VARV_D"] = "🔴 " + df["VARV_D"]
    df["VARP_D"] = df["VARP"].map("{:,.0%}".format).str.replace(",", " ")
    df.loc[df["VARP"] > 0, "VARP_D"] = "+" + df["VARP_D"]
    
    # Plotly: hovertext
    df["HKPI_VALUE"] = df["VALUE"].map("{:,.0f}".format).str.replace(",", " ") + " as of " + df["DATE_D"].astype(str)
    df["HKPI_VAR"] = df["VARV_D"] + " (" + df["VARP_D"] + f") vs {PERIOD_COMP}"
    
    # Filter on entity
    scenario_date = datetime.strptime(scenario, PERIOD_D).strftime(PERIOD)
    df = df[(df.DATE <= scenario_date)]
    
    # Return month rolling
    df.insert(loc=0, column='ENTITY', value=entity)
    df.insert(loc=1, column='SCENARIO', value=scenario)
    return df[-period_rolling:].reset_index(drop=True)

df_barchart = get_trend_barchart(df_master,
                                 "PUBLISHED_DATE",
                                 "VIEWS",
                                 "count",
                                 entity=entities[0],
                                 scenario=scenarios[0])
df_barchart.head(1)
```


```python
def create_data_histo(df, col_date, col_value, agg_value, file_path):
    output_df = pd.DataFrame()
    for entity in entities:
        for scenario in scenarios:
            tmp_df = get_trend_barchart(df,
                                        col_date,
                                        col_value,
                                        agg_value,
                                        entity=entity,
                                        scenario=scenario)
            output_df = pd.concat([output_df, tmp_df]).reset_index(drop=True)
    return output_df
```

##### Posts published


```python
df_1 = create_data_histo(df_master, "PUBLISHED_DATE", "VIEWS", "count", "LINKEDIN_POSTS_PUBLISHED")
print("Rows fetched:", len(df_1))
df_1.head(1)
```

##### Posts views


```python
df_2 = create_data_histo(df_master, "PUBLISHED_DATE", "VIEWS", "sum", "LINKEDIN_POSTS_VIEWS")
print("Rows fetched:", len(df_2))
df_2.head(1)
```

##### Posts likes


```python
df_3 = create_data_histo(df_master, "PUBLISHED_DATE", "LIKES", "sum", "LINKEDIN_POSTS_LIKES")
print("Rows fetched:", len(df_3))
df_3.head(1)
```

##### Posts comments


```python
df_4 = create_data_histo(df_master, "PUBLISHED_DATE", "COMMENTS", "sum", "LINKEDIN_POSTS_COMMENTS")
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
