<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Get_stargazers_from_repository.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Get+stargazers+from+repository:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #stars #naas_drivers #operations #analytics #html #plotly #csv #image #png

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

This notebook enables you to get a dataframe of all the stargazers of a particular Github repository.

## Input

### Imports


```python
import pandas as pd
from datetime import datetime
import plotly.graph_objects as go
from naas_drivers import github
import naas
```

### Setup GitHub
**How to find your personal access token on GitHub?**

- First we need to create a personal access token to get the details of our organization from here: https://github.com/settings/tokens
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API.
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive.
- You can find description of each scope in docs here (https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


```python
# GitHub token
GITHUB_TOKEN = "ENTER_YOUR_GITHUB_TOKEN_HERE" # EXAMPLE : "ghp_fUYP0Z5i29AG4ggX8owctGnHU**********" 

# Github repo on which we want to create issues.
REPO_URL = "ENTER_YOUR_REPO_URL_HERE" # EXAMPLE : "https://github.com/jupyter-naas/awesome-notebooks"
```

### Setup Outputs


```python
# Outputs
name_output = f"awesome-notebooks"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

## Model

### Get stargazers


```python
df_stargazers = github.connect(GITHUB_TOKEN).repos.get_stargazers(REPO_URL)
df_stargazers
```

### Get trend


```python
def get_trend(df,
              date_col_name='STARRED_AT',
              value_col_name="ID",
              date_order='asc'):
    
    # Format date
    df[date_col_name] = pd.to_datetime(df[date_col_name]).dt.strftime("%Y-%m-%d")
    df = df.groupby(date_col_name, as_index=False).agg({value_col_name: "count"})
    d = datetime.now().date()
    d2 = df.loc[df.index[0], date_col_name]
    idx = pd.date_range(d2, d, freq = "D")
    
    df.set_index(date_col_name, drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df[date_col_name] = pd.DatetimeIndex(df.index)
    
    # Calc sum cum
    df["value_cum"] = df.agg({value_col_name: "cumsum"})
    return df.reset_index(drop=True)

df_trend = get_trend(df_stargazers)
df_trend.tail(1)
```

### Plotting a line chart to get trend


```python
def create_linechart(df, date, value, repo_url):
    # Get repo name
    repo_name = repo_url.split("https://github.com/")[-1].split("/")[-1]
    
    # Get last value
    last_value = df.loc[df.index[-1], value]
    
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Scatter(
            x=df[date].to_list(),
            y=df[value].to_list(),
            mode="lines+text",
            line=dict(color="black"),
        )
    )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        title=f"⭐<b> Stars - {repo_name}</b><br><span style='font-size: 13px;'>Total stars as of today: {last_value}</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title="Date",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        yaxis_title='No. of stars',
        yaxis_title_font=dict(family="Arial", size=11, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_trend, "STARRED_AT", "value_cum", REPO_URL)
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
naas.asset.add(image_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
