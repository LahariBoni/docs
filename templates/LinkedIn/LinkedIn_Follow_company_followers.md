<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Follow_company_followers.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Follow+company+followers:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #company #followers #naas_drivers #analytics #automation #csv #html #image #content #plotly

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import linkedin
import pandas as pd
from datetime import datetime
import naas
import plotly.graph_objects as go
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Credentials
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585

# Company URL
COMPANY_URL = "https://www.linkedin.com/company/naas-ai/"
```

### Setup variables


```python
# Inputs
csv_input = "LinkedIn_company_followers.csv"

# Outputs
company_name = COMPANY_URL.strip().split("company/")[-1].split("/")[0]
title = f"{company_name} : LinkedIn company followers"
name_output = "LinkedIn_company_followers_trend"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

### Setup Naas


```python
# Schedule your notebook everyday at 9:00 AM
naas.scheduler.add(cron="0 9 * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get followers from company
**Available columns :**
- FIRSTNAME : First name
- LASTNAME : Last name
- OCCUPATION : Text below the name in the profile page
- PROFILE_PICTURE : Profile picture URL
- PROFILE_URL : Profile URL
- PROFILE_ID : LinkedIn profile id
- PUBLIC_ID : LinkedIn public profile id
- FOLLOWED_AT : Date of following company
- DISTANCE : Distance between your profile


```python
# Get company followers from CSV stored in your local (Returns empty if CSV does not exist)
def get_company_followers(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_followers = get_company_followers(csv_input)
df_followers
```


```python
def get_new_followers(df, input_path):
    # Get all profiles
    profiles = []
    if len(df) > 0:
        profiles = df.PROFILE_ID.unique()
    start = 0
    while True:
        tmp_df = linkedin.connect(LI_AT, JSESSIONID).company.get_followers(COMPANY_URL,
                                                                           start=start,
                                                                           limit=1,
                                                                           sleep=False)
        profile_id = None
        if "PROFILE_ID" in tmp_df.columns:
            profile_id = tmp_df.loc[0, "PROFILE_ID"]
        if profile_id in profiles:
            break
        else:
            df = pd.concat([tmp_df, df])
            df.to_csv(input_path, index=False)
            start += 1
    return df.reset_index(drop=True)

merged_df = get_new_followers(df_followers, csv_input)
merged_df
```

### Prep trend data


```python
def get_trend(df,
              date_col_name=None,
              value_col_name=None,
              date_order='asc'):
    
    # Format date
    df[date_col_name] = pd.to_datetime(df[date_col_name]).dt.strftime("%Y-%m-%d")
    df = df.groupby(date_col_name, as_index=False).agg({value_col_name: "count"})
    d = datetime.now().date()
    d2 = df.loc[df.index[0], date_col_name]
    idx = pd.date_range(d2, d, freq="D")
    
    df.set_index(date_col_name, drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df[date_col_name] = pd.DatetimeIndex(df.index)
    
    # Calc sum cum
    df["VALUE_CUM"] = df.agg({value_col_name: "cumsum"})
    
    df["TEXT"] = (df['VALUE_CUM'].astype(str) + " as of " + df[date_col_name].dt.strftime("%Y-%m-%d") +
                  " (+" + df[value_col_name].astype(str) + " vs yesterday)")
    return df.reset_index(drop=True)

df_trend = get_trend(merged_df, "FOLLOWED_AT", "PROFILE_ID")
df_trend
```

### Create linechart


```python
def create_linechart(df, label, value, text, title):
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value],
            text=df[text],
            hoverinfo="text",
            mode="lines",
            stackgroup="one"
        )
    )
    fig.update_layout(
        title=f"<b>{title}</b><br><span style='font-size: 13px;'>{df.loc[df.index[-1], 'TEXT']}</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_trend, "FOLLOWED_AT", "VALUE_CUM", "TEXT", title)
```

## Output

### Save outputs


```python
df_trend.to_csv(csv_output, index=False)
fig.write_html(html_output)
fig.write_image(image_output)
```

### Save and share CSV with naas


```python
naas.asset.add(csv_output)

#-> to remove your outputs, uncomment the lines and execute the cell
# naas.asset.delete(csv_output)
```

### Save and share HTML with naas


```python
naas.asset.add(html_output, params={"inline": True})

#-> to remove your outputs, uncomment the lines and execute the cell
# naas.asset.delete(html_output)
```

### Save and share image with naas


```python
naas.asset.add(image_output, params={"inline": True})

#-> to remove your outputs, uncomment the lines and execute the cell
# naas.asset.delete(image_output)
```
