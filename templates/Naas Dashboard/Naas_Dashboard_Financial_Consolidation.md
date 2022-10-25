<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas%20Dashboard/Naas_Dashboard_Financial_Consolidation.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+Dashboard+-+Financial+Consolidation:+Error+short+description">🚨 Bug report</a>

**Tags:** #naasdashboard #plotly #dash #naas #asset #automation #analytics #snippet #datavizualisation

**Author:** [Meriem Si](https://www.linkedin.com/in/meriem-si-104236181/)

This notebook enables you to generate a dashboard to follow the financial consolidation.

## Input

### Import libraries


```python
try:
    import dash
except:
    !pip install dash --user
    import dash
from dash import html, dcc, Input, Output, State
try:
    import dash_bootstrap_components as dbc
except:
    !pip install dash_bootstrap_components --user
    import dash_bootstrap_components as dbc
import plotly.graph_objects as go
import plotly.express as px
import os
import pandas as pd
from naas_drivers import gsheet
from dash_bootstrap_components._components.Container import Container
from plotly.subplots import make_subplots
```

### Defining the port of the dashboard


```python
DASH_PORT = 8050
```

### Setup Google Sheets


```python
SPREADSHEET_URL = "https://docs.google.com/spreadsheets/d/127PqDiuz5CXa8tTzhhgUl_ziMYThwZhfWOi86Bo-h-g/edit?usp=sharing"
SHEET_NAME = "KPIS"
SHEET_NAME_2 = "REVENUE"
SHEET_NAME_3 = "OPERATIONAL_EXPENSES"
SHEET_NAME_4 = "EARNING_BEFORE_INTREST_AND_TAXES"
SHEET_NAME_5 = "INCOME_STATEMENT"
```

### Setup Dash App


```python
APP_TITLE = 'Financial Consolidation'
APP_LOGO = "https://images.plot.ly/logo/new-branding/plotly-logomark.png"
```


```python
# Style used for card
CARD_COL_STYLE1= {
    "className": "g-2",
    "xs": 12,
    "sm": 12,
    "md": 12,
    "lg": 2,
    "xl": 2
}

CARD_COL_STYLE2= {
#     "className": "gx-5",
    "xs": 12,
    "sm": 12,
    "md": 12,
    "lg": 12,
    "xl": 12
}

CARD_COL_STYLE3= {
#     "className": "gx-5",
    "xs": 12,
    "sm": 12,
    "md": 12,
    "lg": 6,
    "xl": 6
}
```


```python
# Color used for card
CARD_COLOR = [
    "#163b78",
    "#4c718f",
    "#62a0a6",
    "#64abbd",
    "#45a127",
    "success"
]
BACKGROUND_COLOR = "#f8f9fa"
```

## Model

### Data

#### Dropdown data: Ref Entities / Ref Scenarios


```python
# Lists used for dropdowns intractiveness
entities = [
    "Entity1",
    "Entity2"
]

scenarios = [
    "2022",
    "2021",
    "2020"
]
```

#### Get Kpis data


```python
df_hkpis = gsheet.connect(SPREADSHEET_URL).get(SHEET_NAME)
df_hkpis["VALUE_D"] = df_hkpis["VALUE"].astype(str) + " " + df_hkpis["UNITS"].astype(str)
print("Nb row:", len(df_hkpis))
df_hkpis.head(6)
```

#### Get Revenue data


```python
df_revenue = gsheet.connect(SPREADSHEET_URL).get(SHEET_NAME_2)
print("Nb row:", len(df_revenue))
df_revenue.head(6)
```

#### Get Operational expenses data


```python
df_operational_expenses = gsheet.connect(SPREADSHEET_URL).get(SHEET_NAME_3)
print("Nb row:", len(df_operational_expenses ))
df_operational_expenses.head(6)
```

#### Get Earning before intrest and taxes data


```python
df_earningBIT = gsheet.connect(SPREADSHEET_URL).get(SHEET_NAME_4)
print("Nb row:", len(df_earningBIT))
df_earningBIT.head(6)
```

#### Get Income Statement data


```python
df_income_statement = gsheet.connect(SPREADSHEET_URL).get(SHEET_NAME_5)
df_income_statement["VALUE_D"] = df_income_statement["VALUE"].astype(str) + " " + df_income_statement["UNITS"].astype(str)
print("Nb row:", len(df_income_statement ))
df_income_statement .head(6)
```

### Charts

#### Chart 1 : Revenue


```python
def create_barlinechart(df,
                        label="DATE",
                        value_bar="VALUE_BAR",
                        value_scatter="VAlUE_LINE1",
                        value_scatter_dot="VAlUE_LINE2",
                        xaxis_title=None,
                        yaxis_title_r=None,
                        yaxis_title_l=None):    
    fig = go.Figure()
    # Create figure with secondary y-axis
    fig = make_subplots(specs=[[{"secondary_y": True}]])

    # Add traces
    fig.add_trace(
        go.Bar(
            x=df[label],
            y=df[value_bar],
            name="Growth",
            marker=dict(color="#ADD8E6"),
        ),
        secondary_y=False,
    )
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value_scatter],
            mode="lines",
            name="Revenue",
            line=dict(color="royalblue", width=2.5),
        ),
        secondary_y=False,
    )
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value_scatter_dot],
            name="Target",
            line=dict(color='green', width=4, dash='dot'),
        ),
        secondary_y=True,
    )

    # Add figure title
    fig.update_layout(
        legend=dict(orientation="h"),
        plot_bgcolor="white",
        height=300,
        paper_bgcolor="white",
        xaxis=dict(showgrid=False),
        yaxis=dict(showgrid=False),
        margin=dict(l=20, r=20, t=20, b=20)
    )
    
    fig.update_layout(
        legend=dict(
            orientation="h",
            yanchor="bottom",
            y=1.02,
            xanchor="right",
            x=1
        )
    )

    fig.show()
    return fig


# fig = create_barlinechart(df_revenue_test,
#                           yaxis_title_r=None,
#                           yaxis_title_l=None)
```

#### Chart 2 : Operational Expenses


```python
def stacked_bar(df,
                labels,
                values):
    fig2 = px.bar(df,
                  x="DATE",
                  y="COUNT",
                  color="VALUE",
#                   width=520,
                  height=300,
                  color_discrete_map={
                      'General': '#5ac73c',
                      'Marketing': '#6dabc7',
                      'Sales':'#307a37',
                      'IT':'#30337a'
                  }
                 )
    
    fig2.update_traces(width=0.5)

    fig2.update_layout(yaxis_tickformat = ',',
                       legend_title="",
                       legend=dict(orientation="h"),
                       plot_bgcolor="white",
                       paper_bgcolor="white",
                       xaxis=dict(showgrid=False),
                       yaxis=dict(showgrid=False),
                       margin=dict(l=20, r=20, t=20, b=20)
                      )    
    
    fig2.update_layout(
    legend=dict(
        orientation="h",
        yanchor="bottom",
        y=1.02,
        xanchor="right",
        x=1
    ))
    # Set the visibility OFF
    fig2.update_layout(uniformtext_minsize=8, uniformtext_mode='hide', xaxis_title=None, yaxis_title=None)

    return fig2

# stacked_bar = stacked_bar(df2, None, None)
```

#### Chart 3 : Earnings before Intrest and Taxes


```python
def create_doublelinechart(df,
                        label="DATE",
                        value1="VALUE1",
                        value2="VALUE2",
                        xaxis_title="Months"):    
    fig3 = go.Figure()
    # Add traces

    fig3.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value1],
            mode="lines",
            name="EBIT Actual",
            line=dict(color="#0A66C2", width=2.5),
        ),
    )
    
    fig3.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value2],
            mode="lines",
            name="EBIT Target",
            line=dict(color='green', width=4, dash='dot'),
        ),
    )  

    # Add figure title
    fig3.update_layout(
        legend=dict(orientation="h"),
        plot_bgcolor="white",
        height=300,
#         width=520,
        paper_bgcolor="white",
        xaxis_title=xaxis_title,
        xaxis_title_font=dict(family="Arial", size=10, color="black"),
        xaxis=dict(showgrid=False),
        yaxis=dict(showgrid=False),
        margin=dict(l=20, r=20, t=20, b=40)
    )
    
    # Set the visibility OFF
    fig3.update_layout(uniformtext_minsize=8, uniformtext_mode='hide', xaxis_title=None, yaxis_title=None)
    
    fig3.update_layout(
    legend=dict(
        orientation="h",
        yanchor="bottom",
        y=1.02,
        xanchor="right",
        x=1
    ))
    
    fig3.show()
    return fig3

# doubleline = create_doublelinechart(df3)
```

### Design App

#### Create Dropdown


```python
# Entity's dropdown list
dropdown_entity = dcc.Dropdown(
    id='entity',
    options=[{'label': i, 'value': i} for i in entities],
    placeholder='Entity',
    value=entities[0],
)

# Scenario's dropdown list
dropdown_scenario = dcc.Dropdown(
    id='scenario',
    options=[{'label': i, 'value': i} for i in scenarios],
    placeholder='Scenario',
    value=scenarios[0],
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
                        dbc.Col(html.Img(src=APP_LOGO, height="20")),
                        dbc.Col(dbc.NavbarBrand(APP_TITLE, className="ms-2")),
                    ],
                    align="center",
                    className="g-0",
                ),
            ),
            
            dbc.NavbarToggler(
                id="navbar-toggler",
                n_clicks=0
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
    color="#163b78",
    dark=True,
)
```

#### Create Card


```python
#Function to create card
def create_card_col(uid, card_title, card_paragraph, card_color):
    card = dbc.Col(
        dbc.Card(
            dbc.CardBody(
                [
                    html.P(
                        card_title,
                        className="card-title",
                       style={
                           "font-size": "15px",
                           "padding-bottom": "5px",
                           'text-align':'center',
                           "font-weight": "bold"
                       }
                    ),
                    html.P(
                        card_paragraph,
                        className="card-text",
                        style={
                            "font-size": "16px",
                            'text-align':'center'
                        },
                        id=uid
                    )
                ]
            ),
            color= card_color,
            inverse=True
        ),
        **CARD_COL_STYLE1,
    )
    return card
```

#### Create Chart Card


```python
def create_chart(title, chart_id, CARD_COL_STYLE):
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
                            "font-size": "17px",
                        }
                    ),
                    dcc.Graph(
                        id=chart_id,
                        config={
                            'displayModeBar': True,
                            "staticPlot": True
                        }
                    ),
                ],
            ),
            color=CARD_COLOR,
            inverse=False
        ),
        style={
            "background-color": BACKGROUND_COLOR,
            "display": "inline",
            "padding": "0px"
        },
        **CARD_COL_STYLE
    )
    return card
```

#### Create Income Statment Card


```python
income_statement_card = dbc.Col(
        dbc.Card(
            dbc.CardBody(
                [
                    html.H5(
                        "INCOME STATEMENT",
                        className="card-title",
                        style={"font-weight": "bold", "padding-top": "15px","font-size": "18px"}
                    ),
                    dbc.Row(
                        [
                            dbc.Col(
                                    html.P(
                                        "Revenue",
                                        className="card-text",
                                        style={"font-weight": "bold", 
                                               "padding-top": "20px",
                                               "font-size": "18px"}
                                    )
                            ),
                            dbc.Col(
                                    html.P(
                                        "1,305,507 €",
                                        style={'text-align':'right',"padding-top": "20px","font-size": "18px"},
                                        id="nb1"
                                    )
                            ),
                        ]
                    ),
                    
                    dbc.Row(
                        [
                            dbc.Col(
                                html.P(
                                    "COGS",
                                    className="card-text",
                                    style={"font-weight": "bold","padding-top": "2px","font-size": "18px"}
                                ) 
                            ),
                            dbc.Col(
                                    html.P(
                                        "208,453 €",
                                        style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                        id="nb2"
                                    )
                            ),
                        ]
                    ),
                    html.Hr(
                        style={
                            'borderWidth': "0.5vh",
                            "width": "100%",
                            "color": "black"
                        }
                    ),
                    dbc.Row(
                        [
                            dbc.Col(
                                html.P(
                                    "GROSS PROFIT",
                                    className="card-text",
                                    style={"font-weight": "bold", "padding-top": "2px","font-size": "18px"}
                                )
                            ),
                            dbc.Col(
                                html.P(
                                    "1,097,054 €",
                                    style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                    id="nb3"
                                )
                            )
                        ]
                    ),                              


                    dbc.Row(
                        [
                            dbc.Col(
                                html.P(
                                    "OPEX",
                                    className="card-text",
                                    style={"font-weight": "bold", "padding-top": "2px","font-size": "18px"}
                                )
                            ),
                            dbc.Col(
                                html.P(
                                    "815,306 €",
                                    style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                    id="nb4"
                                )
                            )
                        ]
                    ),

                    dbc.Row(
                        [
                            dbc.Col(
                                html.P(
                                    "Sales",
                                    className="card-text",
                                    style={'margin-left':25, "padding-top": "2px"}
                                )
                            ),
                            dbc.Col(
                                html.P(
                                    "279,886 €",
                                    style={'text-align':'right', "padding-top": "2px"},
                                    id="nb5"
                                )
                            )
                        ]
                    ),
                    
                    dbc.Row(
                        [
                            dbc.Col(
                                html.P(
                                    "Marketing",
                                    className="card-text",
                                    style={'margin-left':25, "padding-top": "2px"}
                                )
                            ),
                            dbc.Col(
                                html.P(
                                    "192,710 €",
                                    style={'text-align':'right', "padding-top": "2px"},
                                    id="nb6"
                                )
                            )
                        ]
                    ),

                dbc.Row(
                    [
                        dbc.Col(
                            html.P(
                                "IT",
                                className="card-text",
                                style={'margin-left':25, "padding-top": "2px"}
                            )
                        ),
                        dbc.Col(
                            html.P(
                                "192,656 €",
                                style={'text-align':'right', "padding-top": "2px"},
                                id="nb7"
                            )
                        )
                    ]
                ),

                dbc.Row(
                    [
                        dbc.Col(
                            html.P(
                                "General & Admin ",
                                className="card-text",
                                style={'margin-left':25, "padding-top": "2px"}
                            )
                        ),
                        dbc.Col(
                            html.P(
                                "150,054 €",
                                style={'text-align':'right', "padding-top": "2px"},
                                id="nb8")
                        )
                    ]
                ),

                dbc.Row(
                    [
                        dbc.Col(
                            html.P(
                                "OTHER INCOME",
                                className="card-text",
                                style={"font-weight": "bold","padding-top": "2px","font-size": "18px"}
                            )
                        ),
                        dbc.Col(
                            html.P("2,130 €",
                                   style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                   id="nb9"
                                  )
                        )
                    ]
                ),

                dbc.Row(
                    [
                        dbc.Col(
                            html.P(
                                "OTHER EXPENSES",
                                className="card-text",
                                style={"font-weight": "bold", "padding-top": "2px","font-size": "18px"}
                            )
                        ),
                        dbc.Col(
                            html.P(
                                "51,195 €",
                                style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                id="nb10"
                            )
                        )
                    ]
                ),

                html.Hr(
                    style={
                        'borderWidth': "0.5vh",
                        "width": "100%",
                        "color": "blak"
                    }
                ),

                dbc.Row(
                    [
                        dbc.Col(
                            html.P(
                                "EBIT",
                                className="card-text",
                                style={"font-weight": "bold", "padding-top": "2px","font-size": "18px"}
                            )
                        ),
                        
                        dbc.Col(
                            html.P(
                                "232,684 €",
                                style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                id="nb11"
                            )
                        ),
                    ]
                ),

                dbc.Row(
                    [
                        dbc.Col(
                            html.P(
                                "Intrest and Tax",
                                className="card-text",
                                style={'margin-left':25, "padding-top": "2px","font-size": "18px"}
                            )
                        ),
                        dbc.Col(
                            html.P(
                                "1,305,507 €",
                                style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                id="nb12"
                            )
                        )
                    ]
                ),

                html.Hr(style={'borderWidth': "0.5vh",
                               "width": "100%",
                               "color": "blak"}
                       ),

                dbc.Row(
                    [
                        dbc.Col(
                            html.P(
                                "NET PROFIT",
                                className="card-text",
                                style={"font-weight": "bold", "padding-top": "2px","font-size": "18px"}
                            )
                        ),
                        dbc.Col(
                            html.P(
                                "194,440 €",
                                style={'text-align':'right', "padding-top": "2px","font-size": "18px"},
                                id="nb13"
                            )
                        )
                    ]
                )            
            ]
        ),
    ),
#     className="gx-5",
    xs=12, sm=12, md=12, lg=4, xl=4,
)
```

#### Create app layout


```python
app = dash.Dash(requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
                external_stylesheets=[dbc.themes.BOOTSTRAP],
                meta_tags=[{'name': 'viewport','content': 'width=device-width, initial-scale=1.0'}]
                )

#app = dash.Dash() if you are not in Naas
app.title = APP_TITLE
app.layout = html.Div(
    [
        #Navbar
        navbar,

        #HKPIS
        dbc.Row(
            [
                create_card_col("card1", "REVENUE TARGET", "85%", CARD_COLOR[0]),
                create_card_col("card2", "COGS RATIO", "78%", CARD_COLOR[1]),
                create_card_col("card3", "GROSS MARGIN", "55%", CARD_COLOR[2]),
                create_card_col("card4", "OPEX RATIO", "85%", CARD_COLOR[3]),
                create_card_col("card5", "EBIT MARGIN", "52%", CARD_COLOR[4]),
                create_card_col("card6", "NET MARGIN", "95%", CARD_COLOR[5]),
            ],
            className="g-0 d-flex align-items-center",
        ),

        #SPACE
        html.Br(),

        #CHARTS
        dbc.Row(
            [
                #Column 1: Charts
                dbc.Col(
                    [
                        #Row 1: 1 Charts
                        dbc.Row(
                            create_chart("REVENUE", "fig1", CARD_COL_STYLE2),
                        ),
                        #Row 2: 2 Charts
                        dbc.Row(
                            [
                                create_chart("OPERATIONAL EXPENSES", "fig2", CARD_COL_STYLE3),
                                create_chart("EARNINGS BEFORE INTREST AND TAXES", "fig3", CARD_COL_STYLE3)
                            ]
                        )
                    ],
                    xs=12, sm=12, md=12, lg=8, xl=8,
                ),
                #Column 2: Income Statement Card 
                income_statement_card,
            ]
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

# add callback to filter data in cards and charts
@app.callback(
    [
        Output('card1', 'children'),
        Output('card2', 'children'),
        Output('card3', 'children'),
        Output('card4', 'children'),
        Output('card5', 'children'),
        Output('card6', 'children'),
        Output('fig1', 'figure'),
        Output('fig2', 'figure'),
        Output('fig3', 'figure'),
        Output('nb1', 'children'),
        Output('nb2', 'children'),
        Output('nb3', 'children'),
        Output('nb4', 'children'),
        Output('nb5', 'children'),
        Output('nb6', 'children'),
        Output('nb7', 'children'),
        Output('nb8', 'children'),
        Output('nb9', 'children'),
        Output('nb10', 'children'),
        Output('nb11', 'children'),
        Output('nb12', 'children'),
        Output('nb13', 'children'),
    ],
    [
        Input('entity', 'value'),
        Input('scenario', 'value')
    ]
    
)

def multi_outputs(entity, scenario):
    if entity is None and scenario is None:
        raise PreventUpdate
        
    # Get HKPIs dataframe
    dfhkpis = df_hkpis.copy()
    dfhkpis = dfhkpis[(dfhkpis["ENTITY"] == entity) &
                      (dfhkpis["SCENARIO"].astype(str) == scenario)].reset_index(drop=True)
    
    # Return card data
    card1 = dfhkpis.loc[dfhkpis["LABEL"] == "Revenue Target", "VALUE_D"].values[0]
    card2 = dfhkpis.loc[dfhkpis["LABEL"] == "COGS Ratio", "VALUE_D"].values[0]
    card3 = dfhkpis.loc[dfhkpis["LABEL"] == "Gross Profit Margin", "VALUE_D"].values[0]
    card4 = dfhkpis.loc[dfhkpis["LABEL"] == "Opex Ratio", "VALUE_D"].values[0]
    card5 = dfhkpis.loc[dfhkpis["LABEL"] == "Ebit Margin", "VALUE_D"].values[0]
    card6 = dfhkpis.loc[dfhkpis["LABEL"] == "Net Profit Margin", "VALUE_D"].values[0]
    
    
    # Get Revenue graph dataframe
    dfrevenue = df_revenue.copy()
    dfrevenue = dfrevenue[(dfrevenue["ENTITY"] == entity) &
                      (dfrevenue["SCENARIO"].astype(str) == scenario)].reset_index(drop=True)
    
    # Get Operational expenses graph dataframe
    dfoperationalexpenses = df_operational_expenses.copy()
    dfoperationalexpenses = dfoperationalexpenses[(dfoperationalexpenses["ENTITY"] == entity) &
                      (dfoperationalexpenses["SCENARIO"].astype(str) == scenario)].reset_index(drop=True)
    
    # Get Earning before intrest and taxes graph dataframe
    dfearningBIT = df_earningBIT.copy()
    dfearningBIT = dfearningBIT [(dfearningBIT ["ENTITY"] == entity) &
                      (dfearningBIT ["SCENARIO"].astype(str) == scenario)].reset_index(drop=True)
    
    # Create graphs
    fig1 = create_barlinechart(dfrevenue, yaxis_title_r=None, yaxis_title_l=None)
    fig2 = stacked_bar(dfoperationalexpenses, None, None)
    fig3 = create_doublelinechart(dfearningBIT)

    
        # Get Earning before intrest and taxes graph dataframe
    dfincome_statement = df_income_statement.copy()
    dfincome_statement = dfincome_statement[(dfincome_statement ["ENTITY"] == entity) &
                      (dfincome_statement ["SCENARIO"].astype(str) == scenario)].reset_index(drop=True)
    
    
    # Return values of income statement card data
    nb1 = dfincome_statement.loc[dfincome_statement["LABEL"] == "Revenue", "VALUE_D"].values[0]
    nb2 = dfincome_statement.loc[dfincome_statement["LABEL"] == "COGS", "VALUE_D"].values[0]
    nb3 = dfincome_statement.loc[dfincome_statement["LABEL"] == "Gross Profit", "VALUE_D"].values[0]
    nb4 = dfincome_statement.loc[dfincome_statement["LABEL"] == "Opex", "VALUE_D"].values[0]
    nb5 = dfincome_statement.loc[dfincome_statement["LABEL"] == "Sales", "VALUE_D"].values[0]
    nb6 = dfincome_statement.loc[dfincome_statement["LABEL"] == "Marketing", "VALUE_D"].values[0]
    nb7 = dfincome_statement.loc[dfincome_statement["LABEL"] == "IT", "VALUE_D"].values[0]
    nb8 = dfincome_statement.loc[dfincome_statement["LABEL"] == "General & Admin", "VALUE_D"].values[0]
    nb9 = dfincome_statement.loc[dfincome_statement["LABEL"] == "OTHER INCOME", "VALUE_D"].values[0]
    nb10 = dfincome_statement.loc[dfincome_statement["LABEL"] == "OTHER EXPENSES", "VALUE_D"].values[0]
    nb11 = dfincome_statement.loc[dfincome_statement["LABEL"] == "EBIT", "VALUE_D"].values[0]
    nb12 = dfincome_statement.loc[dfincome_statement["LABEL"] == "Intrest and Tax", "VALUE_D"].values[0]
    nb13 = dfincome_statement.loc[dfincome_statement["LABEL"] == "NET PROFIT", "VALUE_D"].values[0]
    return card1, card2, card3, card4, card5, card6, fig1, fig2, fig3, nb1, nb2, nb3, nb4, nb5, nb6, nb7, nb8, nb9, nb10, nb11, nb12, nb13
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```


```python

```
