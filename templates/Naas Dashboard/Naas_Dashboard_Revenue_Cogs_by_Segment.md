<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas%20Dashboard/Naas_Dashboard_Revenue_Cogs_by_Segment.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+Dashboard+-+Revenue+Cogs+by+Segment:+Error+short+description">🚨 Bug report</a>

**Tags:** #naasdashboard #plotly #dash #naas #asset #automation #analytics #snippet #datavizualisation

**Author:** [Fernando Chavez Osuna](https://www.linkedin.com/in/fernando-chavez-osuna-1a420a181)

This notebook enables you to generate a dashboard about Revenue & COGS by segment.

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
import naas_drivers
from naas_drivers import gsheet
from dash_bootstrap_components._components.Container import Container
import pandas as pd
```

### Setup APP


```python
DASH_PORT = 8050
APP_TITLE = 'Revenue & COGS by Segment'
APP_LOGO = "https://pic.onlinewebfonts.com/svg/img_542879.png"
```

### Setup Google Sheets

- Share your Google Sheets spreadsheet with our service account : 🔗 naas-share@naas-gsheets.iam.gserviceaccount.com


```python
SPREADSHEET_URL = "https://docs.google.com/spreadsheets/d/1SmXeixAxpmDCso7zRBgflg1405P-GMDiSrjnvTFzdy0/edit?usp=sharing"
SHEET_NAME = "SALES_COGS"
SHEET_NAME_2 = "PRODUCT_SALES"
```

## Model

### Data

#### Ref Entities


```python
entities = [
    "All Regions",
    "EMEA",
    "APAC",
    "AMERICAS"
]
```

#### Ref Scenarios


```python
scenarios = [
    "2022",
    "2021",
    "2020",
]
```

#### Get Gross Sales and COGS Data


```python
sales_cogs_df = gsheet.connect(SPREADSHEET_URL).get(SHEET_NAME)
sales_cogs_df
```

#### Get Product Sales per Country Data


```python
product_sales = gsheet.connect(SPREADSHEET_URL).get(SHEET_NAME_2)
product_sales = product_sales.groupby(['SCENARIO', 'COUNTRY','LABEL'])['VALUE'].sum().reset_index()
product_sales = product_sales.pivot(index=['SCENARIO', 'LABEL'], columns='COUNTRY', values="VALUE")
product_sales.loc[:, "SCENARIO"] = product_sales.index.get_level_values(0)
product_sales.loc[:, "LABEL"] = product_sales.index.get_level_values(1)
product_sales = product_sales.reset_index(drop=True)
product_sales
```

### Graphs

#### Gross Sales & COGS by Segment - Grouped Bar Chart


```python
def create_bar_chart(df, x, y):
    df_gs = df.copy()
    df_gs = df_gs[df_gs.UNITS == 'Gross_Sales']
    
    df_cogs = df.copy()
    df_cogs = df_cogs[df_cogs.UNITS == 'COGS']
    bar_chart = go.Figure(data=[
        go.Bar(name='Gross Sales', x=df_gs[x], y=df_gs[y], marker_color='rgb(58,100,152)'),
        go.Bar(name='COGS', x=df_cogs[x], y=df_cogs[y], marker_color='rgb(240,163,83)')
    ])
    
    bar_chart.update_layout(
        title_text='<b>Gross Sales and COGS by Segment',
        title_x=0.5,
        
        # Change the bar mode
        barmode = 'group',
        
        xaxis=dict(
            title='<b>Segment',
            titlefont_size=14,
            tickfont_size=14,
            ticks='outside',
            showline=True,
            linewidth=1,
            linecolor='black'
        ),
        
        yaxis=dict(
            title='<b>Gross Sales & COGS',
            titlefont_size=14,
            tickfont_size=14,
            ticks='outside',
            showline=True,
            linewidth=1,
            linecolor='black'
        ),
        
        legend=dict(
            title="<b>All Measures",
            x=1,
            y=1,
            bgcolor='rgba(255, 255, 255, 0)',
            bordercolor='rgba(255, 255, 255, 0)'
        ),
        
        plot_bgcolor='rgba(0,0,0,0)',
    )
    return bar_chart

df = sales_cogs_df[sales_cogs_df["SCENARIO"].astype(str) == scenarios[0]]
bar_chart = create_bar_chart(df, "ENTITY", "VALUE")
bar_chart
```

#### Sales by Segment - Pie Chart


```python
def create_pie_chart(df, labels, values):
    pie_chart = go.Figure(
        data=[go.Pie(
            labels=df[labels],
            values=df[values],
            hole=0.5)
             ]
    )

    pie_chart.update_traces(
        textinfo='none', 
        marker = dict(colors=px.colors.qualitative.Vivid)
    )
    
    pie_chart.update_layout(
        title='<b>Sales by Segment', 
        title_x=0.5,
    )
    return pie_chart

df = sales_cogs_df[sales_cogs_df["SCENARIO"].astype(str) == scenarios[0]]
pie_chart = create_pie_chart(df, "ENTITY", "VALUE")
pie_chart
```

#### Gross Sales by Country and Product - Heatmap Chart


```python
def create_heatmap_chart(df):
    labels = df["LABEL"].unique()
    df = df.drop(["SCENARIO", "LABEL"], axis=1)
    heat_chart = go.Figure(
                    data=go.Heatmap(
                            z=df,
                            x=df.columns,
                            y=labels,
                            colorscale='greens',
                            colorbar={"title": '<b>Gross Sales'}))

    heat_chart.update_layout(
        
        title_text='<b>Gross Sales by Country and Product',
        title_x=0.9,
        
        xaxis=dict(
            title='<b>Country',
            titlefont_size=14,
            tickfont_size=14,
        ),
        
        yaxis=dict(
            title='<b>Product',
            titlefont_size=14,
            tickfont_size=14,
            ticks='outside',
            showline=True,
            linewidth=1,
            linecolor='black',
        ),
    )
    return heat_chart

df = product_sales[product_sales["SCENARIO"].astype(str) == scenarios[0]]
heat_chart = create_heatmap_chart(df)
heat_chart
```

### Dash App

#### Create dropdown object


```python
dropdown_entity = dcc.Dropdown(
    id='entity',
    options=[{'label': i, 'value': i} for i in entities],
    placeholder='Entity',
    value=entities[0],
)

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
                        dbc.Col(html.Img(src=APP_LOGO, height="30px")),
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
    color="#808080",
    dark=True,
)
```

#### Create App Layout 


```python
app = dash.Dash(requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
                external_stylesheets=[dbc.themes.BOOTSTRAP], 
                meta_tags=[{'name': 'viewport','content': 'width=device-width, initial-scale=1.0'}])   
#app = dash.Dash() if you are not in Naas

app.title = APP_TITLE
app.layout = html.Div(
    [
        #Navbar
        navbar,
        
        #Charts
        dbc.Row(
            [
                dbc.Col(
                    dcc.Graph(id='fig1',
                              figure=bar_chart,
                              className="h-100"),
                    xs=12,
                    sm=12,
                    md=12,
                    lg=6,
                    xl=6
                ),
                dbc.Col(
                    [
                        dbc.Row(
                            [
                                dcc.Graph(id='fig2',
                                          figure=pie_chart),
                            ]
                        ),
                        dbc.Row(
                            [
                                dcc.Graph(id='fig3',
                                          figure=heat_chart),
                            ]
                        )
                    ],
                    xs=12,
                    sm=12,
                    md=12,
                    lg=6,
                    xl=6
                )
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

# add callback to filter data in charts
@app.callback(
    [
        Output('fig1', 'figure'),
        Output('fig2', 'figure'),
        Output('fig3', 'figure'),
    ],
    [
        Input('entity', 'value'),
        Input('scenario', 'value')
    ]
)

def multi_outputs(entity, scenario):
    # Get Gross Sales & COGS graph dataframe
    sales_cogs = sales_cogs_df.copy()
    sales_cogs = sales_cogs[
        (sales_cogs["SCENARIO"].astype(str) == scenario)
    ].reset_index(drop=True)

    # Get Product Sales per Country graph dataframe
    country_product_sales = product_sales.copy()
    country_product_sales = country_product_sales[
        (country_product_sales["SCENARIO"].astype(str) == scenario)
    ].reset_index(drop=True)

#         # Create graphs
    fig1 = create_bar_chart(sales_cogs, "ENTITY", "VALUE")
    fig2 = create_pie_chart(sales_cogs, "ENTITY", "VALUE") 
    fig3 = create_heatmap_chart(country_product_sales)
    return fig1, fig2, fig3
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```


```python

```
