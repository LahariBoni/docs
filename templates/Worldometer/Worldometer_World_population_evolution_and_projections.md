<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Worldometer/Worldometer_World_population_evolution_and_projections.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Worldometer+-+World+population+evolution+and+projections:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldometer #opendata #population #snippet #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import libraries


```python
import pandas as pd
import plotly.express as px
from bs4 import BeautifulSoup
import requests
```

### Data to scrap tables


```python
DATA_URLS = ["https://www.worldometers.info/world-population/world-population-by-year/",
    "https://www.worldometers.info/world-population/world-population-projections/"
    ]

TABLE_COLS = ['Year',
    'World Population',
    'YearlyChange',
    'NetChange',
    'Density(P/Km²)',
    'UrbanPop',
    'UrbanPop %']
```

## Model

### Functions to scrap tables on several sites, and merge them


```python
# Generic functions

def scrap_table(url, table_cloumns):
    page = requests.get(url)
    soup = BeautifulSoup(page.text, 'html.parser')
    dfs = pd.read_html(page.text)

    for df in dfs:
        if df.columns.to_list() == table_cloumns:
            return df
    return None

def merge_tables_from_urls(urls, table_columns):
    table = None
    for url in urls:
        new_value = scrap_table(url, table_columns)
        if new_value is not None:
            if table is None:
                table = new_value
            else:
                table = table.append(new_value)
    return table
```

### Print table


```python
table
```

### Create function to display graph


```python
def create_graph(x_label, y_label, table, title="", graph_type=px.line):
    fig = graph_type(table, x=x_label, y=y_label, title=title)
    fig.show()
```


```python
# Print population graph from year to year
def display_population_graph(table, x_from=None, x_to=None, graph_type=px.line):
    x_label = TABLE_COLS[0]
    y_label = TABLE_COLS[1]
    if x_from is not None:
        table = table[table.Year >= x_from]
    if x_to is not None:
        table = table[table.Year <= x_to]
    title = f"{y_label} by {x_label}, between {table[x_label].to_list()[-1]} and {table[x_label].to_list()[0]}"
    create_graph(x_label, y_label, table, title, graph_type)
```

### Fetch tables, sort the result and remove duplicate data


```python
table = merge_tables_from_urls(DATA_URLS, TABLE_COLS)

table = table.sort_values(by=[TABLE_COLS[0]], ascending=False)

table.drop_duplicates(subset=TABLE_COLS[0], keep="first", inplace=True)
```

## Output

### Display the graph between -5000 and 2100


```python
chart1 = display_population_graph(table)
```

### Display the graph between 1800 and 2020


```python
display_population_graph(table, x_from=1800, x_to=2020)
```

### Display the graph between 2000 and 2100


```python
display_population_graph(table, x_from=2000, x_to=2100)
```

### Display a barchart between 2000 and 2100 

The graph type can be change by passing a graph function as 'graph_type' (graph_type=px.line, etc)


```python
display_population_graph(table, x_from=1950, x_to=2100, graph_type=px.bar)
```
