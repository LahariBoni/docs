<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dash/Dash_Create_Navbar.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dash+-+Create+Navbar:+Error+short+description">🚨 Bug report</a>

**Tags:** #dashboard #plotly #dash #naas #asset #analytics #navbar #bootstrap

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

A simple app demonstrating how to manually construct a navbar with a customised layout using the Navbar component and the supporting Nav, NavItem, NavLink, NavbarBrand, and NavbarToggler components.

Requires dash-bootstrap-components 0.3.0 or later

<u>Reference:</u> https://github.com/facultyai/dash-bootstrap-components/blob/main/examples/python/advanced-component-usage/navbars.py

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
from dash import html, Input, Output, State
```

### Setup Variables


```python
DASH_PORT = 8050
PLOTLY_LOGO = "https://images.plot.ly/logo/new-branding/plotly-logomark.png"
```

## Model

### Utils


```python
# make a reuseable navitem for the different examples
nav_item = dbc.NavItem(dbc.NavLink("Link", href="#"))

# make a reuseable dropdown for the different examples
dropdown = dbc.DropdownMenu(
    children=[
        dbc.DropdownMenuItem("Entry 1"),
        dbc.DropdownMenuItem("Entry 2"),
        dbc.DropdownMenuItem(divider=True),
        dbc.DropdownMenuItem("Entry 3"),
    ],
    nav=True,
    in_navbar=True,
    label="Menu",
)
```

### Navbar Simple


```python
# this is the default navbar style created by the NavbarSimple component
default = dbc.NavbarSimple(
    children=[nav_item, dropdown],
    brand="NavbarSimple",
    brand_href="#",
    sticky="top",
    className="mb-5",
)
```

### Navbar Custom


```python
# here's how you can recreate the same thing using Navbar
# (see also required callback at the end of the file)
custom_default = dbc.Navbar(
    dbc.Container(
        [
            dbc.NavbarBrand("Custom default", href="#"),
            dbc.NavbarToggler(id="navbar-toggler1"),
            dbc.Collapse(
                dbc.Nav(
                    [nav_item, dropdown], className="ms-auto", navbar=True
                ),
                id="navbar-collapse1",
                navbar=True,
                is_open=False,
            ),
        ]
    ),
    className="mb-5",
)
```

### Navbar Custom with logo


```python
# this example that adds a logo to the navbar brand
logo = dbc.Navbar(
    dbc.Container(
        [
            html.A(
                # Use row and col to control vertical alignment of logo / brand
                dbc.Row(
                    [
                        dbc.Col(html.Img(src=PLOTLY_LOGO, height="30px")),
                        dbc.Col(dbc.NavbarBrand("Logo", className="ms-2")),
                    ],
                    align="center",
                    className="g-0",
                ),
                href="https://plotly.com",
                style={"textDecoration": "none"},
            ),
            dbc.NavbarToggler(id="navbar-toggler2", n_clicks=0),
            dbc.Collapse(
                dbc.Nav(
                    [nav_item, dropdown],
                    className="ms-auto",
                    navbar=True,
                ),
                id="navbar-collapse2",
                navbar=True,
                is_open=False,
            ),
        ],
    ),
    color="dark",
    dark=True,
    className="mb-5",
)
```

### Navbar Custom with search


```python
# this example has a search bar and button instead of navitems / dropdowns
search_navbar = dbc.Navbar(
    dbc.Container(
        [
            dbc.NavbarBrand("Search", href="#"),
            dbc.NavbarToggler(id="navbar-toggler3"),
            dbc.Collapse(
                dbc.Row(
                    [
                        dbc.Col(
                            dbc.Input(type="search", placeholder="Search")
                        ),
                        dbc.Col(
                            dbc.Button(
                                "Search", color="primary", className="ms-2"
                            ),
                            # set width of button column to auto to allow
                            # search box to take up remaining space.
                            width="auto",
                        ),
                    ],
                    # add a top margin to make things look nice when the navbar
                    # isn't expanded (mt-3) remove the margin on medium or
                    # larger screens (mt-md-0) when the navbar is expanded.
                    # keep button and search box on same row (flex-nowrap).
                    # align everything on the right with left margin (ms-auto).
                    className="g-0 ms-auto flex-nowrap mt-3 mt-md-0",
                    align="center",
                ),
                id="navbar-collapse3",
                navbar=True,
                is_open=False,
            ),
        ]
    ),
    className="mb-5",
)
```

### Navbar Custom Dashboard


```python
# custom navbar based on https://getbootstrap.com/docs/4.1/examples/dashboard/
dashboard = dbc.Navbar(
    dbc.Container(
        [
            dbc.Col(dbc.NavbarBrand("Dashboard", href="#"), sm=3, md=2),
            dbc.Col(dbc.Input(type="search", placeholder="Search here")),
            dbc.Col(
                dbc.Nav(
                    dbc.Container(dbc.NavItem(dbc.NavLink("Sign out"))),
                    navbar=True,
                ),
                width="auto",
            ),
        ],
    ),
    color="dark",
    dark=True,
)

```

### Create app layout


```python
app = dash.Dash(
    requests_pathname_prefix=f'/user/{os.environ.get("JUPYTERHUB_USER")}/proxy/{DASH_PORT}/', 
    external_stylesheets=[dbc.themes.BOOTSTRAP],
    meta_tags=[{'name':'viewport', 'content':'width=device-width, initial-scale=1.0'}]
) 

app.layout = html.Div(
    [
        default,
        custom_default,
        logo,
        search_navbar,
        dashboard
    ]
)

# we use a callback to toggle the collapse on small screens
def toggle_navbar_collapse(n, is_open):
    if n:
        return not is_open
    return is_open


# the same function (toggle_navbar_collapse) is used in all three callbacks
for i in [1, 2, 3]:
    app.callback(
        Output(f"navbar-collapse{i}", "is_open"),
        [Input(f"navbar-toggler{i}", "n_clicks")],
        [State(f"navbar-collapse{i}", "is_open")],
    )(toggle_navbar_collapse)
```

## Output

### Generate URL and show logs


```python
if __name__ == '__main__':
    app.run_server(proxy=f"http://127.0.0.1:{DASH_PORT}::https://app.naas.ai")
```
