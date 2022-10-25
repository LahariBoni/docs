<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Follow_content_published_monthly.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Follow+content+published+monthly:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #html #plotly #csv #image #content #analytics #dependency

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook follows the evolution of content published on LinkedIn.

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
```

### Setup Variables


```python
# Input
csv_input = f"LINKEDIN_PROFILE_POSTS.csv" # CSV path with your posts stats generated with 'LinkedIn_Get_profile_posts_stats.ipynb' or 'LinkedIn_Get_company_posts_stats.ipynb'
TITLE = "Posts" # Chart title
COL_VALUE = "POST_URL" # Column to sum

# Outputs
name_output = "LINKEDIN_FOLLOW_CONTENT_PUBLISHED_MONTHLY"
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

### Create trend dataframe


```python
DATE_FORMAT = "%Y-%m-%d"
PERIOD = "%Y-%m"
PERIOD_TEXT = "This month"

def get_trend(df_init,
              col_date,
              col_value,
              agg_value,
              month_rolling=12):
    # Init variable
    df = df_init.copy()
    
    # Groupby period
    df[col_date] = pd.to_datetime(df[col_date].str[:-6]).dt.strftime(DATE_FORMAT)
    df = df.groupby(col_date, as_index=False).agg({col_value: agg_value})
    
    # Rename column
    to_rename = {
        col_date: "DATE",
        col_value: "VALUE"
    }
    df = df.rename(columns=to_rename)
    # Reindex value
    d = datetime.now().date()
    d2 = df.loc[df.index[0], "DATE"]
    idx = pd.date_range(d2, d, freq = "D")    
    df.set_index("DATE", drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df["DATE"] = pd.DatetimeIndex(df.index)

    # Groupby month
    df["DATE"] = pd.to_datetime(df["DATE"], format=DATE_FORMAT).dt.strftime(PERIOD)
    df = df.groupby("DATE", as_index=False).agg({"VALUE": "sum"})

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
    
    # Plotly: Date display
    df["DATE_D"] = pd.to_datetime(df["DATE"], format="%Y-%m").dt.strftime("%b %Y")
    
    # Plotly: Value display
    df["VALUE_D"] = "<b><span style='font-family: Arial;'>" + df["VALUE"].map("{:,.0f}".format).str.replace(",", " ") + "</span></b>"
    
    # Plotly: Variation display
    df["VARV_D"] = df["VARV"].map("{:,.0f}".format).str.replace(",", " ")
    df.loc[df["VARV"] >= 0, "VARV_D"] = "+" + df["VARV_D"]
    df["VARP_D"] = df["VARP"].map("{:,.0%}".format).str.replace(",", " ")
    df.loc[df["VARP"] >= 0, "VARP_D"] = "+" + df["VARP_D"]
    
    # Plotly: hovertext
    df["TEXT"] = ("<b><span style='font-size: 14px;'>" + df["DATE_D"].astype(str) + ": " + df["VALUE_D"] + "</span></b><br>"
                  "<span style='font-size: 12px;'>" + f"{PERIOD_TEXT}: " + df["VARV_D"] + " (" + df["VARP_D"] + ")</span>")
    
    # Return month rolling
    return df[-month_rolling:].reset_index(drop=True)

df_trend = get_trend(df_posts,
                     col_date="PUBLISHED_DATE",
                     col_value=COL_VALUE,
                     agg_value="count")
df_trend
```

## Output

### Display linechart


```python
LOGO = "https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/LinkedIn_logo_initials.png/800px-LinkedIn_logo_initials.png" # Chart logo
COLOR = "#1293d2" # Chart primary color

def create_barchart(df,
                    label="DATE_D",
                    value="VALUE",
                    value_d="VALUE_D",
                    text="TEXT"):
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Bar(
            x=df[label],
            y=df[value],
            text=df[value_d],
            textposition="outside",
            hoverinfo="text",
            hovertext=df[text],
            marker=dict(color=COLOR)
        )
    )
    # Add logo
    fig.add_layout_image(
        dict(
            source=LOGO,
            xref="paper",
            yref="paper",
            x=0.01,
            y=1.045,
            sizex=0.12,
            sizey=0.12,
            xanchor="right",
            yanchor="bottom"
        )
    )
    fig.update_traces(showlegend=False)
    # Plotly: Create title
    total_value = "{:,.0f}".format(df[value].sum()).replace(",", " ")
    varv = df.loc[df.index[-1], "VARV"]
    varp = df.loc[df.index[-1], "VARP"]
    varv_d = "{:,.0f}".format(varv).replace(",", " ")
    varp_d = "{:,.0%}".format(varp).replace(",", " ")
    if varv >= 0:
        varv_d = f"+{varv_d}"
        varp_d = f"+{varp_d}"
    title = f"<b><span style='font-size: 20px;'>{TITLE}</span></b><br><span style='font-size: 18px;'>Total: {total_value} | {PERIOD_TEXT}: {varv_d} ({varp_d})</span>"
    fig.update_layout(
        title=title,
        title_x=0.09,
        title_font=dict(family="Arial", color="black"),
        paper_bgcolor="#ffffff",
        plot_bgcolor="#ffffff",
        width=1200,
        height=600,
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_barchart(df_trend)
```

### Save and share your csv file


```python
# Save your dataframe in CSV
df_trend.to_csv(csv_output, index=False)

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
