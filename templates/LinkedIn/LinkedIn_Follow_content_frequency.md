<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Follow_content_frequency.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Follow+content+frequency:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #html #plotly #csv #image #content #analytics #automation

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook enables to follow the number of content views monthly on LinkedIn on your company page, cumulated since first activity.

<div class="alert alert-info" role="info" style="margin: 10px">
    <p><b>Requirements:</b><br>To run this notebook, you must have already run one of this notebook to get your company post stats in CSV:
        <li>LinkedIn_Get_posts_stats_from_company.ipynb</li>
        <li>LinkedIn_Update_metrics_from_company_posts_in_Notion_content_calendar.ipynb</li>
</p>
</div>

## Input

### Import libraries


```python
from naas_drivers import linkedin
from os import path
import naas
import pandas as pd
from datetime import datetime
import plotly.graph_objects as go
from plotly.subplots import make_subplots
```

### Setup Variables


```python
# Inputs
csv_input = "LINKEDIN_POSTS_XXXXXX.csv"

# Outputs
name_output = "Linkedin_Follow_number_of_company_content_views_monthly"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"

# Period
PERIOD = "%Y-%m"

# Custom chart
primary_color = "#D3D3D3"
secundary_color = "#5C5C5C"
company_logo = "https://media-exp2.licdn.com/dms/image/C560BAQEOzG0TtTclXw/company-logo_400_400/0/1606695523917?e=1662595200&v=beta&t=2k73kaJGBZ4i1OtFmy-0Vms6uXu7bLd2AJbhrx_D4AA"
```

### Setup Naas


```python
naas.dependency.add()

#-> Uncomment the line below to remove your dependency
# naas.dependency.delete()
```

## Model

### Get your posts from CSV
All your posts will be stored in CSV.


```python
def read_csv(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_posts = read_csv(csv_input)
df_posts
```

### Get dataframe trend


```python
def get_trend(df_init, col_date, col_value, label):
    # Init variable
    df = df_init.copy()
    
    # By period
    df[col_value] = df[col_value].astype(float)
    df[col_date] = pd.to_datetime(df[col_date].str[:-6]).dt.strftime(PERIOD)
    df = df.groupby(col_date, as_index=False).agg({col_value: "sum"})
    
    # Calc sum cum
    to_rename = {
        col_date: "DATE",
        col_value: "VARV"
    }
    df = df.rename(columns=to_rename)
    df["VALUE"] = df.agg({"VARV": "cumsum"})
    
    # Add last month
    if df.loc[df.index[-1], "DATE"] != datetime.now().strftime(PERIOD):
        tmp_df = df[-1:].reset_index(drop=True)
        tmp_df.loc[0, "DATE"] = datetime.now().strftime(PERIOD)
        tmp_df.loc[0, "VARV"] = 0
        df = pd.concat([df, tmp_df]).reset_index(drop=True)
        
    # Calc order
    df["ORDER"] = pd.to_datetime(df["DATE"]).dt.strftime("%Y%m%d")
    
    # Filter data
    df = df[df.ORDER.astype(int) >= 20210228]
    df = df.drop("ORDER", axis=1)
    
    # Calc var
    df["VALUE_COMP"] = df["VALUE"] - df["VARV"]
    df["VARP"] = df["VARV"] / abs(df["VALUE_COMP"])
    df["LABEL"] = label
    df = df[["DATE", "LABEL", "VALUE", "VALUE_COMP", "VARV", "VARP"]].reset_index(drop=True)
    
    # Prep data
    df["VALUE_D"] = df["VALUE"].map("{:,.0f}".format).str.replace(",", " ")
    df["VARV_D"] = df["VARV"].map("{:,.0f}".format).str.replace(",", " ")
    df.loc[df["VARV"] >= 0, "VARV_D"] = "+" + df["VARV_D"]
    df["VARP_D"] = df["VARP"].map("{:,.0%}".format).str.replace(",", " ")
    df.loc[df["VARP"] >= 0, "VARP_D"] = "+" + df["VARP_D"]
    
    # Create hovertext
    df["TEXT"] = ("<b><span style='font-size: 14px;'>" + df["LABEL"].astype(str)  + " " + df["DATE"].astype(str) + " : " + df["VALUE_D"] + "</span></b><br>"
                  "<span style='font-size: 12px;'>" + df["VARV_D"] + " this month (" + df["VARP_D"] + ")</span>")
    df["TITLE"] = ("<b><span style='font-size: 20px;'>" + df["LABEL"].astype(str)  + " " + df["DATE"].astype(str)  + " : " + df["VALUE_D"] + "</span></b><br>"
                  "<span style='font-size: 18px;'>" + df["VARV_D"] + " this month (" + df["VARP_D"] + ")</span>")
    for index, row in df.iterrows():
        if index > 0:
            n = df.loc[df.index[index], "VARV"]
            n_1 = df.loc[df.index[index-1], "VARV"]
            df.loc[df.index[index], "VARV_COMP"] = n_1
            df.loc[df.index[index], "VARV_VARV"] = n - n_1
            if n_1 > 0:
                df.loc[df.index[index], "VARP_VARV"] = (n - n_1) / abs(n_1)
    return df.reset_index(drop=True)

df_actual = get_trend(df_posts,
                      col_date="PUBLISHED_DATE",
                      col_value="VIEWS",
                      label="LinkedIn Posts views")

df_actual#.tail(5)
```

### Plotting a barlinechart to get trend


```python
def create_barlinechart(df,
                        label="DATE",
                        value="VALUE",
                        varv="VARV",
                        varp="VARP",
                        text="TEXT",
                        title="TITLE",
                        xaxis_title="Weeks",
                        yaxis_title_r=None,
                        yaxis_title_l=None,
                        primary_color=None,
                        secundary_color=None,
                        company_logo=None,):    
    # Create figure with secondary y-axis
    fig = make_subplots(specs=[[{"secondary_y": True}]])

    # Add traces
    fig.add_trace(
        go.Bar(
            x=df[label],
            y=df[varv],
            hoverinfo="text",
            text=df["VARV_D"],
            hovertext=df[text],
            marker=dict(color=primary_color),
        ),
        secondary_y=False,
    )
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value],
            mode="lines",
            hoverinfo="text",
            text=df["VALUE_D"],
            hovertext=df[text],
            line=dict(color=secundary_color, width=3),
        ),
        secondary_y=True,
    )
    # Add figure title
    title_text = text
    for col in df.columns:
        if col == title:
            title_text = title
    fig.update_layout(
        title=df.loc[df.index[-1], title_text],
        title_x=0.1,
        title_font=dict(family="Arial", size=20, color="black"),
        legend=None,
        plot_bgcolor="#ffffff",
        paper_bgcolor="#ffffff",
#         width=1200,
#         height=800,
        autosize=False,
        margin=dict(
            l=100,
            b=100,
        ),
        margin_pad=10,
        xaxis_title=xaxis_title,
        xaxis_title_font=dict(family="Arial", size=14, color="black"),
    )
    # Add logo
    fig.add_layout_image(
        dict(
            source=company_logo,
            xref="paper",
            yref="paper",
            x=0.01,
            y=1,
            sizex=0.12,
            sizey=0.12,
            xanchor="right",
            yanchor="bottom"
        )
    )

    # Set y-axes titles
    fig.update_yaxes(
        title_text=yaxis_title_r,
        title_font=dict(family="Arial", size=14, color="black"),
        secondary_y=False
    )
    fig.update_yaxes(
        title_text=yaxis_title_l,
        title_font=dict(family="Arial", size=14, color="black"),
        secondary_y=True
    )
    fig.update_traces(showlegend=False)
    fig.show()
    return fig

fig = create_barlinechart(df_actual,
                          primary_color=primary_color,
                          secundary_color=secundary_color,
                          company_logo=company_logo,
                          xaxis_title="Months",
                          yaxis_title_r="New views",
                          yaxis_title_l="Total views")
```

## Output


### Save and share your csv file


```python
# Save your dataframe in CSV
df_actual.to_csv(csv_output, index=False)

# Share output with naas
csv_link = naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(csv_output)
```

### Save and share your graph in HTML


```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
html_link = naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```

### Save and share your graph in image


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
image_link = naas.asset.add(image_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
