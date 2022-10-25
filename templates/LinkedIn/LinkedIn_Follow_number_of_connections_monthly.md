<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Follow_number_of_connections_monthly.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Follow+number+of+connections+monthly:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #network #connections #naas_drivers #analytics #csv #html #image #content #plotly

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This template gives you an overview of your LinkedIn connections over the last 12 months, making it easier to track your efforts in connecting with your LinkedIn network.<br>

## Input

### Import libraries


```python
from naas_drivers import linkedin
import pandas as pd
from datetime import datetime
import naas
import plotly.graph_objects as go
from plotly.subplots import make_subplots
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn cookies
LI_AT = "ENTER_YOUR_COOKIE_HERE" # EXAMPLE : "AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2"
JSESSIONID = "ENTER_YOUR_JSESSIONID_HERE" # EXAMPLE : "ajax:8379907400220387585"
```

### Setup Outputs
Create CSV to store your posts stats.<br>
PS: This CSV could be used in others LinkedIn templates.


```python
# Inputs
csv_input = f"LINKEDIN_CONNECTIONS.csv"
no_rolling_months = 12 # number of rolling months

# Outputs
name_output = f"LINKEDIN_CONNECTIONS_MONTHLY_{str(no_rolling_months)}R"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

### Setup Naas scheduler


```python
# Schedule your notebook everyday at 9:00 AM
naas.scheduler.add(cron="0 9 * * *")

# this notebook will run each week until de-scheduled
# to de-schedule this notebook, simply run the following command: 
# naas.scheduler.delete()
```

## Model

### Get your connections from CSV
All your connections will be stored in CSV.


```python
def get_csv(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df
```


```python
df_connections = get_csv(csv_input)
df_connections
```

### Update last connections
It will get the last connections posts from LinkedIn API.<br>
PS: On the first execution all connections will be retrieved. The execution could take some times depending of the size of your network.


```python
def update_connections(df,
                       csv_input):
    # Init variables
    profiles = []
    new_connections = 0
    
    # Get existing connections if not get all connections
    if len(df) > 0:
        profiles = df["PROFILE_ID"].unique()
        
        # Get new connections
        update = True
        
        while update:
            start = 0
            df_new = linkedin.connect(LI_AT, JSESSIONID).network.get_connections(start=start,
                                                                                 count=100,
                                                                                 limit=100)
            df_new["DATE_EXTRACT"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            new_profiles = df_new["PROFILE_ID"].unique()
            for i, p in enumerate(new_profiles):
                if p in profiles:
                    update = False
                    df_new = df_new[:i]
                    print(i, "new connections")
                    break
            start += 100
            new_connections += i
            df = pd.concat([df_new, df]).drop_duplicates("PROFILE_ID", keep="first").reset_index(drop=True)
            df.to_csv(csv_input, index=False)
    else:
        df = linkedin.connect(LI_AT, JSESSIONID).network.get_connections(limit=-1)
        df["DATE_EXTRACT"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        
    # Concat, save database in CSV and dependency in production
    if new_connections > 0:
        print(f"🚀 {new_connections} new connections found in LinkedIn.")
    else:
        print(f"🛑 No new connections found in LinkedIn.")
    df.to_csv(csv_input, index=False)
    naas.dependency.add(csv_input)
    
    print("Total LinkedIn connections:", len(df))
    return df.reset_index(drop=True)

df_connections = update_connections(df_connections, csv_input)
df_connections
```

### Get trend dataframe
Create dataframe with number of LinkedIn views cumulated by monthly with variation


```python
def get_trend(df_init,
              col_date,
              col_value,
              label,
              months_limit=12):
    # Init variable
    df = df_init.copy()
    
    # By week
    df[col_date] = pd.to_datetime(df[col_date].str[:-6]).dt.strftime("%Y-%m")
    df = df.groupby([col_date], as_index=False).agg({col_value: "count"})
    
    # Calc sum cum
    df["VALUE"] = df.agg({col_value: "cumsum"})
    
    # Cleaning
    to_rename = {
        col_date: "DATE",
        col_value: "VARV"
    }
    df = df.rename(columns=to_rename)
    
    # Calc var
    df["VALUE_COMP"] = df["VALUE"] - df["VARV"]
    df["VARP"] = df["VARV"] / abs(df["VALUE_COMP"])
    df["LABEL"] = label
    df = df[["DATE", "LABEL", "VALUE", "VALUE_COMP", "VARV", "VARP"]].reset_index(drop=True)
    
    # Prep data
    df["VALUE_D"] = df["VALUE"].map("{:,.0f}".format).str.replace(",", " ")
    df["VARV_D"] = df["VARV"].map("{:,.0f}".format).str.replace(",", " ")
    df.loc[df["VARV"] > 0, "VARV_D"] = "+" + df["VARV_D"]
    df["VARP_D"] = df["VARP"].map("{:,.0%}".format).str.replace(",", " ")
    df.loc[df["VARP"] > 0, "VARP_D"] = "+" + df["VARP_D"]
    
    # Create hovertext
    df["TEXT"] = ("<b>" + df["LABEL"] + " as of " + df["DATE"] + ": " + df["VALUE_D"] + "</b> <span style='font-size: 14px;'>(" + df["VARP_D"] + " growth)</span><br>"
                  "<span style='font-size: 14px;'>" + df["VARV_D"] + " connections this month</span>")
    for index, row in df.iterrows():
        if index > 0:
            n = df.loc[df.index[index], "VARV"]
            n_1 = df.loc[df.index[index-1], "VARV"]
            df.loc[df.index[index], "VARV_COMP"] = n_1
            df.loc[df.index[index], "VARV_VARV"] = n - n_1
            df.loc[df.index[index], "VARP_VARV"] = 0
            if n_1 > 0:
                df.loc[df.index[index], "VARP_VARV"] = (n - n_1) / abs(n_1)
    return df[-months_limit:].reset_index(drop=True)

df_trend = get_trend(df_connections,
                     col_date="CREATED_AT",
                     col_value="PROFILE_ID",
                     label="LinkedIn connections",
                     months_limit=no_rolling_months)

df_trend.tail(5)
```

### Create barlinechart using Plotly


```python
def create_barlinechart(df,
                        label="DATE",
                        value="VALUE",
                        varv="VARV",
                        varp="VARP",
                        text="TEXT",
                        xaxis_title="Months",
                        yaxis_title_r=None,
                        yaxis_title_l=None):    
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
            marker=dict(color="#ADD8E6"),
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
            line=dict(color="#0A66C2", width=2.5),
        ),
        secondary_y=True,
    )

    # Add figure title
    fig.update_layout(
        title=df.loc[df.index[-1], text],
        title_font=dict(family="Arial", size=18, color="black"),
        legend=None,
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title=xaxis_title,
        xaxis_title_font=dict(family="Arial", size=10, color="black"),
    )

    # Set y-axes titles
    fig.update_yaxes(
        title_text=yaxis_title_r,
        title_font=dict(family="Arial", size=10, color="black"),
        secondary_y=False
    )
    fig.update_yaxes(
        title_text=yaxis_title_l,
        title_font=dict(family="Arial", size=10, color="black"),
        secondary_y=True
    )
    fig.update_traces(showlegend=False)
    fig.show()
    return fig

fig = create_barlinechart(df_trend,
                          yaxis_title_r="Monthly connections",
                          yaxis_title_l="Total connections")
```

## Output

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
