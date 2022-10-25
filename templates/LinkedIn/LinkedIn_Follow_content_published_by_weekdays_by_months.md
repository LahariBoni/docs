<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Follow_content_published_by_weekdays_by_months.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Follow+content+published+by+weekdays+by+months:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #html #plotly #csv #image #content #analytics #dependency

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook displays the number of content published on your LinkedIn posts by weekdays and by months.

<div class="alert alert-info" role="info" style="margin: 10px">
<b>Requirements:</b><br>
To run this notebook, you must have already run <b>LinkedIn_Get_profile_posts_stats.ipynb</b> or <b>LinkedIn_Get_company_posts_stats.ipynb</b> to get your post stats in CSV.<br>
</div>

## Input

### Import libraries


```python
import naas
import pandas as pd
from datetime import datetime
import plotly.graph_objects as go
from pandas.tseries.offsets import MonthEnd
import calendar
from dateutil.relativedelta import relativedelta
```

### Setup Variables


```python
# Input
csv_input = f"LINKEDIN_PROFILE_POSTS.csv" # CSV path with your posts stats generated with 'LinkedIn_Get_profile_posts_stats.ipynb' or 'LinkedIn_Get_company_posts_stats.ipynb'
TITLE = "Content published frequency" # Chart title

# Outputs
name_output = "LINKEDIN_FOLLOW_CONTENT_VIEWS_REACH"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

### Setup Naas dependency


```python
naas.dependency.add()

#-> Uncomment the line below to remove your dependency
# naas.dependency.delete()
```

## Model

### Get your posts
Get posts feed from CSV stored in your local (Returns empty if CSV does not exist)


```python
def read_csv(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_posts = read_csv(csv_input)
print("✅ Posts fetched:", len(df_posts))
df_posts.head(1)
```

### Get frequency


```python
MONTH_ROLLING = 12

def get_frequency(df_init,
                  col_date,
                  x_axis,
                  y_axis,
                  col_value,
                  type_value
                 ):
    # Init variable
    df = df_init.copy()
    
    # Setup date column and create X and Y axis analysis
    df[col_date] = pd.to_datetime(df[col_date].str[:18])
    df["X_AXIS"] = df[col_date].dt.strftime(x_axis)
    df["Y_AXIS"] = df[col_date].dt.strftime(y_axis)
    df = df.rename(columns={col_value: "VALUE"})
    
    # Groupby
    to_group = [
        "X_AXIS",
        "Y_AXIS",
    ]
    df = df.groupby(to_group, as_index=False).agg({"VALUE": type_value})

    # Create empty value
    d = datetime.now() + MonthEnd(1)
    d2 = df["X_AXIS"].min()
    idx = pd.date_range(d2, d, freq="m", normalize=True)
    for x in idx:
        x = x.strftime("%Y-%m")
        data = [
            {"X_AXIS": x, "Y_AXIS": "1", "VALUE": 0},
            {"X_AXIS": x, "Y_AXIS": "2", "VALUE": 0},
            {"X_AXIS": x, "Y_AXIS": "3", "VALUE": 0},
            {"X_AXIS": x, "Y_AXIS": "4", "VALUE": 0},
            {"X_AXIS": x, "Y_AXIS": "5", "VALUE": 0},
            {"X_AXIS": x, "Y_AXIS": "6", "VALUE": 0},
            {"X_AXIS": x, "Y_AXIS": "7", "VALUE": 0},
        ]
        tmp_df = pd.DataFrame(data)
        df = pd.concat([tmp_df, df])
        
    # Group by with empty values
    df = df.groupby(to_group, as_index=False).agg({"VALUE": "sum"})
        
    # Sort values
    month_min = datetime.now() + relativedelta(months=-MONTH_ROLLING)
    df = df[df["X_AXIS"].str.replace("-", "").astype(int) >= int(month_min.strftime("%Y%m"))]
    df = df.sort_values(by=["X_AXIS", "Y_AXIS"], ascending=[True, False])
    return df.reset_index(drop=True)

df_plotly = get_frequency(df_posts,
                          col_date="PUBLISHED_DATE",
                          x_axis="%Y-%m",
                          y_axis="%u",
                          col_value="ACTIVITY_ID",
                          type_value="count")
df_plotly#.tail(10)
```

### Plot Heatmap


```python
LOGO = "https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/LinkedIn_logo_initials.png/800px-LinkedIn_logo_initials.png" # Chart logo
COLOR = "#1293d2" # Chart primary color

def create_heatmap(df,
                   x_value="X_AXIS",
                   y_value="Y_AXIS",
                   z_value="VALUE",
                   x_format="%H",
                   x_format_d="%H",
                   text="views",
                   ):
    
    # Add display values
    df["X_AXIS_D"] = pd.to_datetime(df[x_value], format=x_format).dt.strftime(x_format_d)
    df["Y_AXIS_D"] = df.apply(lambda row: calendar.day_name[int(row[y_value]) - 1], axis=1)
    df["TEXT"] = df[z_value].astype(str) + f" {text} on " + df["Y_AXIS_D"] + "s, " + df["X_AXIS_D"]

    # Create graph data
    x = sorted(df[x_value].unique().tolist())
    y = sorted(df[y_value].unique().tolist(), reverse=True)
    def get_values(df, y, value):
        values = []
        for i in y:
            tmp = df[df[y_value] == i].reset_index(drop=True)
            data = tmp[value].tolist()
            values.append(data)
        return values
    z = get_values(df, y, z_value)
    hovertext = get_values(df, y, "TEXT")
    
    # Colors
    colors = [
        [0.00, "#e7f4fa"],
        [0.01, "#b7def1"],
        [0.25, "#88c9e8"],
        [0.50, "#59b3df"],
        [1.00, "#1293d2"]
    ]

    # Create fig
    fig = go.Figure(
        data=go.Heatmap(
            x=df["X_AXIS_D"].unique().tolist(),
            y=df["Y_AXIS_D"].unique().tolist(),
            z=z,
            text=hovertext,
            hoverinfo="text",
            type='heatmap',
            colorscale=colors,
            hoverongaps=False,
        )
    )
    fig.add_layout_image(
        dict(
            source=LOGO,
            xref="paper",
            yref="paper",
            x=-0.01,
            y=1.045,
            sizex=0.12,
            sizey=0.12,
            xanchor="right",
            yanchor="bottom"
        )
    )
    fig.update_traces(xgap=10,
                      ygap=10,
                      selector=dict(type='heatmap'),
                      showscale=False)
    total_value = "{:,.0f}".format(df[z_value].sum()).replace(",", " ")
    fig.update_layout(
        title = f"<b><span style='font-size: 20px;'>{TITLE}</span></b><br><span style='font-size: 18px;'>Total {text}: {total_value}</span>",
        title_x=0.08,
        title_font=dict(family="Arial", size=20, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=600,
        yaxis_scaleanchor="x"
    )
    fig.show()
    return fig

fig = create_heatmap(df_plotly,
                     x_format="%Y-%m",
                     x_format_d="%b %Y",
                     text="contents published")
```

## Output


### Save and share your csv file


```python
# Save your dataframe in CSV
df_plotly.to_csv(csv_output, index=False)

# Share output with naas
naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(csv_output)
```

### Save and share your graph in HTML


```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```

### Save and share your graph in image


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
naas.asset.add(image_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
