<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas%20Dashboard/Naas_Dashboard_Social_Media_KPIs_ScoreCard.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+Dashboard+-+Social+Media+KPIs+ScoreCard:+Error+short+description">🚨 Bug report</a>

**Tags:** #naasdashboard #plotly #dash #naas #asset #automation #analytics #snippet #datavizualisation

**Author:** [Ismail CHIHAB](https://www.linkedin.com/in/ismail-chihab-4b0a04202/)

This notebook creates a dashboard to follow your social media kpis on LinkedIn, YouTube, Instagram and Twitter.

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
import naas
from naas_drivers import gsheet
from dash import html, dash_table
from dash.dependencies import Input, Output, State
from dash import dcc
import pandas as pd
import os
```

### Setup Dash App


```python
DASH_PORT = 8050
APP_TITLE = "Social Media KPIs"

# Logo used in Dash App -> to display it in HTML you must create an naas asset using naas.asset.add()
linkedin_logo = "https://public.naas.ai/aXNtYWlsY2hpaGFiNzEtNDBnbWFpbC0yRWNvbQ==/asset/3fee2f8a3764e507215a3fcff091270e46eec4d696c1cb50b870d6e03264"
instagram_logo = "https://public.naas.ai/aXNtYWlsY2hpaGFiNzEtNDBnbWFpbC0yRWNvbQ==/asset/063fd6d5f4bcf0333e24bc9df3e27ff0ac94d87cb3f49deb618e136a2820.png"
twitter_logo = "https://public.naas.ai/aXNtYWlsY2hpaGFiNzEtNDBnbWFpbC0yRWNvbQ==/asset/d558bf1a83a9016a68d2bc47c488de17668d5710f3cab70f01aa7e366f05.png"
youtube_logo = "https://public.naas.ai/aXNtYWlsY2hpaGFiNzEtNDBnbWFpbC0yRWNvbQ==/asset/1c8d463751a136a2cdead2a0e8c663b1fa19bb623e1d2b8874c17cf6bb96.png"
```

### Setup Google Sheets


```python
spreadsheet_url = "https://docs.google.com/spreadsheets/d/1sJgHWhQIj5R11XeNAA4gEj426T1OFS15gbu1DQMvlfk/edit#gid=1460434917"
sh_linkedin = "001"
sh_youtube = "002"
sh_instagram = "003"
sh_twitter = "004"
```

## Model

### Data

#### Get LinkedIn data


```python
linkedin_df = gsheet.connect(spreadsheet_url).get(sheet_name=sh_linkedin) 
print("☑️ Number of rows:", len(linkedin_df))
linkedin_df.head(1)
```

#### Get YouTube data


```python
youtube_df = gsheet.connect(spreadsheet_url).get(sheet_name=sh_youtube) 
print("☑️ Number of rows:", len(youtube_df))
youtube_df.head(1)
```

#### Get Instagram data


```python
instagram_df = gsheet.connect(spreadsheet_url).get(sheet_name=sh_instagram) 
print("☑️ Number of rows:", len(instagram_df))
instagram_df.head(1)
```

#### Get Twitter data


```python
twitter_df = gsheet.connect(spreadsheet_url).get(sheet_name=sh_twitter)
print("☑️ Number of rows:", len(twitter_df))
twitter_df.head(1)
```

#### Pivot dataframe to create table with rows and columns


```python
def update_data(df_init):
    # Drop duplicates
    df = df_init.copy()
    df = df.drop_duplicates()

    # Pivot 
    df = pd.pivot(df, index=['ROWS', 'SCENARIO'], values='VALUE', columns='COLUMNS')
    df.loc[:, "SCENARIO"] = df.index.get_level_values(1)
    df.loc[:, "ROWS"] = df.index.get_level_values(0)
    df = df.reset_index(drop=True)
    
    
    #Re format values with a comma in columns to target and to previous
    to_target=[]       
    for e in df['To Target']:
        if "," in str(e):
            e = e.replace(',', '.')
            
        to_target.append(e)
    df["To Target"] = to_target
    
    to_previous=[]       
    for e in df['To Previous']:
        if "," in str(e):
            e = e.replace(',', '.')
            
        to_previous.append(e)
    df["To Previous"] = to_previous
    
    
    #Re arranging the columns
    df = df.reindex( columns = ['ROWS','Actual','Target','To Target','Prev Period', 'To Previous','SCENARIO'])
    df.reset_index(drop=True)
    df.rename(columns={'ROWS': ''}, inplace=True)
    return df 
```


```python
linkedin_dff = update_data(linkedin_df)
youtube_dff = update_data(youtube_df)
instagram_dff = update_data(instagram_df)
twitter_dff = update_data(twitter_df)
```

#### Create columns list


```python
table_columns = [{'id': c, 'name': c} for c in linkedin_dff.iloc[:, 0:6].columns]
table_columns
```

### Dash App

#### Create Dropdowns


```python
entities = [
    "All Platforms",
    "Linkedin",
    "Youtube",
    "Instagram",
    "Twitter"
]

scenarios = [
    "2022",
    "2021",
    "2020",
]

dropdown_entity = dcc.Dropdown(
    id='entity',
    options=[{'label': i, 'value': i} for i in entities],
    placeholder='Entity',
    value=entities[0],
    style={
        "text-align": "center",
    }
)

dropdown_scenario = dcc.Dropdown(
    id='scenario',
    options=[{'label': i, 'value': i} for i in scenarios],
    placeholder='Scenario',
    value=scenarios[0],
    style={
        "text-align": "center",
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
                        #dbc.Col(html.Img(src=APP_LOGO, height="30px")),
                        dbc.Col(dbc.NavbarBrand(APP_TITLE, className="ms-2")),
                    ],
                    align="center",
                    className="g-0",
                ),
                href="https://mobile.twitter.com/ws_room/photo",
                style={"textDecoration": "none"},
            ),
            dbc.NavbarToggler(id="navbar-toggler", n_clicks=0),
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
    color="dark",
    dark=True,
)
```

#### Create Tables header


```python
def inner_navbar(platform_name, logo, brand_hex_color): 
    inner_navbar = dbc.Navbar(
            dbc.Container(
                [
                    html.A(
                        # Use row and col to control vertical alignment of logo / brand
                        dbc.Row(
                            [
                                dbc.Col(html.Img(src=logo, height="30px")),
                                dbc.Col(dbc.NavbarBrand(platform_name, className="ms-2")),
                            ],
                            align="center",
                            className="g-0",
                        ),
                        style={"textDecoration": "none"},
                    )
                ]
            ),
            color=str(brand_hex_color),
            dark=True,
        )
    return inner_navbar
```


```python
LinkedIn_navbar =  inner_navbar("LinkedIn", linkedin_logo, "#0E76A8")
Instagram_navbar =  inner_navbar("Instagram", instagram_logo, "#ed5a9c")
Twitter_navbar =  inner_navbar("Twitter", twitter_logo, "#6cd5f7")
Youtube_navbar =  inner_navbar("Youtube", youtube_logo, "#ed494a")
```

#### Create Tables


```python
def create_table(df, table_id):    
    table = dash_table.DataTable( 
        id=table_id,
        data=df.to_dict('records'),
        columns=table_columns,
        
        # Style
        style_data={
             'whiteSpace': 'normal',
             'height':'16px',
             'lineHeight': '13px',
             'border': '7px solid white'
         },
        style_cell_conditional=[
            {
                'if': {'column_id': ''},
                'textAlign': 'left',
                'width': '150px'
            },
            {
                'if': {'column_id': 'Actual'},
             'width': '150px'
            }, 
             {
                 'if': {'column_id': 'Target'},
              'width': '150px'
             },
             {
                 'if': {'column_id': 'To Target'},
              'width': '150px'
             },
             {
                 'if': {'column_id': 'Prev Period'},
              'width': '150px'
             },
             {
                 'if': {'column_id': 'To Previous'},
              'width': '150px'
             },
        ],

        style_data_conditional=([
             {
              'if': {
                  'column_id': 'Target', 
              },  
              'backgroundColor': '#f9f9f9',
              'color': 'black'
            },
            {
              'if': {
                  'column_id': 'Prev Period', 
              },  
              'backgroundColor': '#f9f9f9',
              'color': 'black'
            },
            {
              'if': {
                  'filter_query': '{To Target} < 0',
                  'column_id': 'To Target', 
              },  
              'backgroundColor': '#ebc7c3',
              'color': '#cf8076'
            },
            {
              'if': {
                  'filter_query': '{To Target} >= 0',
                  'column_id': 'To Target', 
              },  
              'backgroundColor': '#d5e6d1',
              'color': 'rgba(146,170,107,255)'
            },
            {
              'if': {
                  'filter_query': '{To Previous} < 0',
                  'column_id': 'To Previous', 
              },  
              'backgroundColor': '#ebc7c3',
              'color': '#cf8076'
            },
            {
              'if': {
                  'filter_query': '{To Previous} >= 0',
                  'column_id': 'To Previous', 
              },  
              'backgroundColor': '#d5e6d1',
              'color': '#92aa6b'
            },

        ]),

        style_table={
            'overflowX':'scroll',
        },
        style_cell={
            'padding': '5px',
            'textAlign': 'center',
            'font-family':'sans-serif'

        },
        style_header={
            'backgroundColor': 'white',
            'fontWeight': 'bold',
            'border': '7px solid white'

        },
        #style_as_list_view=True,
    )
    return table
```

#### Create Cards with headers and table


```python
responsive = {  'xs':12,
                'sm':12,
                'md':12,
                'lg':6,
                'xl':6 }
def card_table(navbar, df, uid, responsive):
    card = dbc.Col(
        dbc.Card(
            dbc.CardBody(
                [
                    navbar,
                    create_table(df, uid),

                ]
            ),
        ),
        **responsive
    )
    return card
```

#### Create Layout


```python
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    meta_tags=[
        {
            'name':'viewport',
            'content': 'width=device-width, initial-scale=1.0'
        }
    ]
)

app.title = APP_TITLE
app.layout = html.Div(
    [
        #Navbar:
        navbar,
        
        #The page content:
        dbc.Container(
            id = 'main_container'
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

# add callback to display interactive data
@app.callback(
    Output('main_container', 'children'), 
    [
        Input('entity', 'value'),
        Input('scenario', 'value')
    ]
    
)

def multi_outputs(entity, scenario):
    if entity is None or scenario is None:
        raise PreventUpdate
    
    # Get dataframes
    lk_df = linkedin_dff.copy()
    yt_df = youtube_dff.copy()
    ig_df = instagram_dff.copy()
    tit_df = twitter_dff.copy()
    
    
    rows=['Followers', 'Impressions', 'Link Clicks', 'Engagement', 'Engagement Rate', 'Avg order Value', 'Avg Time to Conversion']
    # Filter tables
    if str(scenario) == '2021':
        lk_df = lk_df[lk_df['SCENARIO'] == 2021]
        yt_df = yt_df[yt_df['SCENARIO'] == 2021]
        ig_df = ig_df[ig_df['SCENARIO'] == 2021]
        tit_df = tit_df[tit_df['SCENARIO'] == 2021]
        
        #lk_df_a = lk_df_a[lk_df_a['SCENARIO'] == 2021]
        
    elif str(scenario) == '2022':
        lk_df = lk_df[lk_df['SCENARIO'] == 2022]
        yt_df = yt_df[yt_df['SCENARIO'] == 2022]
        ig_df = ig_df[ig_df['SCENARIO'] == 2022]
        tit_df = tit_df[tit_df['SCENARIO'] == 2022]
        
        #lk_df_a = lk_df[lk_df['SCENARIO'] == 2022]
        
    elif str(scenario) =='2020':
        lk_df = lk_df[lk_df['SCENARIO'] == 2020]
        yt_df = yt_df[yt_df['SCENARIO'] == 2020]
        ig_df = ig_df[ig_df['SCENARIO'] == 2020]
        tit_df = tit_df[tit_df['SCENARIO'] == 2020]
        
    
    # Rearrange row order
    lk_df[''] = pd.Categorical(lk_df[''], categories=rows, ordered=True)
    lk_df = lk_df.sort_values('', axis=0)
    
    yt_df[''] = pd.Categorical(yt_df[''], categories=rows, ordered=True)
    yt_df = lk_df.sort_values('', axis=0)
    
    ig_df[''] = pd.Categorical(ig_df[''], categories=rows, ordered=True)
    ig_df = lk_df.sort_values('', axis=0)
    
    tit_df[''] = pd.Categorical(tit_df[''], categories=rows, ordered=True)
    tit_df = lk_df.sort_values('', axis=0)
    
    
    # Drop the SCENARIO column
    lk_df.drop(['SCENARIO'], axis=1, inplace=True) 
    yt_df.drop(['SCENARIO'], axis=1, inplace=True)  
    ig_df.drop(['SCENARIO'], axis=1, inplace=True) 
    tit_df.drop(['SCENARIO'], axis=1, inplace=True)
    

    # Switch between pages:
    responsive = {  
                 'xs':12,
                 'sm':12,
                 'md':12,
                 'lg':12,
                 'xl':12 }
    children=[]
    children.append(html.Br())
    if str(entity) == 'Linkedin':
        children.append(dbc.Row([card_table(LinkedIn_navbar, lk_df, "linkedin_table", responsive)])) 
    if str(entity) == 'Youtube':
        children.append(dbc.Row([card_table(Youtube_navbar, yt_df, "youtube_table", responsive)]))
    if str(entity) == 'Instagram':
        children.append(dbc.Row([card_table(Instagram_navbar, ig_df, "instagram_table", responsive)]))
    if str(entity) == 'Twitter':
        children.append(dbc.Row([card_table(Twitter_navbar, tit_df, "twitter_table", responsive)]))
    if str(entity) == 'All Platforms':
        responsive = {  
                 'xs':12,
                 'sm':12,
                 'md':6,
                 'lg':6,
                 'xl':6 }
        children.append(
            dbc.Row(
                    [
                        card_table(LinkedIn_navbar, lk_df, "linkedin_table", responsive),
                        card_table(Instagram_navbar, ig_df, "instagram_table", responsive)
                    ]
                )
        )
        children.append(
            dbc.Row(
                    [
                        card_table(Twitter_navbar, tit_df, "twitter_table", responsive),
                        card_table(Youtube_navbar, yt_df, "youtube_table", responsive)
                    ]
                )
        )
        
    return children
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```
