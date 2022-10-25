<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Track_notebooks_created_over_time.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Track+notebooks+created+over+time:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #repos #commits #notebooks #operations #analytics #html #plotly #csv #image #png

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input


```python
import pandas as pd
import requests
import os
from naas_drivers import github
import plotly.graph_objects as go
import pydash as _pd
from urllib.parse import urlencode
from datetime import datetime, timedelta
import naas
```

## Setup Github
**How to find your personal access token on Github?**

- First we need to create a personal access token to get the details of our organization from here: https://github.com/settings/tokens
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API.
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive.
- You can find description of each scope in docs here (https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


```python
# Inputs
REPO_URL = "https://github.com/jupyter-naas/awesome-notebooks"
GITHUB_TOKEN = "ghp_COJiJEU4cQR4rjsxxxxx"

# Outputs
chart_title = "Notebooks created since naas launch"
name_output = f"awesome-notebooks"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

## Model

### Get commits


```python
df_commits = github.connect(GITHUB_TOKEN).repos.get_commits(REPO_URL)
print("Total commits:", len(df_commits))
df_commits.head(1)
```

### Filter on merge PR and date


```python
def get_commits_merge(df):
    df_pr = df[(df.MESSAGE.str[:5] == "Merge") & 
               (df.VERIFICATION_STATUS == True)].reset_index(drop=True)
    print("Total Merged PR:", len(df_pr))

    df_pr["DATE"] = df_pr["AUTHOR_DATE"].dt.strftime("%Y-%m-%d")
    df_pr = df_pr[["DATE", "COMMITTER_NAME", "ID"]].drop_duplicates("DATE").reset_index(drop=True)
    print("Total Merged PR (unique date):", len(df_pr))
    return df_pr

df_pr = get_commits_merge(df_commits)
df_pr.head(1)
```

### Get notebooks for each commits


```python
def get_notebooks(commit_id):
    notebooks = []
    headers = {'Authorization': f'token {GITHUB_TOKEN}'}
    url = f"https://api.github.com/repos/jupyter-naas/awesome-notebooks/git/trees/{commit_id}?recursive=1"
    res = requests.get(url, headers=headers)
    try:
        res.raise_for_status()
    except requests.HTTPError as e:
        raise(e)
    res_json = res.json()

    trees = res_json.get("tree")
    for file in trees:
        if ".github" not in file.get("path") and ".gitignore" not in file.get("path") and "/" in file.get("path"):
            if file.get("path").endswith(".ipynb"):
                temp = file.get("path").split("/")
                if temp == -1:
                    data = {
                        "root": None,
                        "subdir": file.get("path")
                    }
                    notebooks.append(data)
                else:
                    last_folder = ""
                    file_name = temp[-1]
                    temp.pop()
                    for folder in temp:
                        last_folder += "/" + folder
                    root = last_folder[1:]
                    data = {
                        "root": root,
                        "subdir": file_name
                    }
                    notebooks.append(data)
    return pd.DataFrame(notebooks)

def tracks_notebooks(df):
    df_tracks = pd.DataFrame()

    for _, row in df.iterrows():
        date = row.DATE
        commit_id = row.ID
        try:
            df_track = get_notebooks(commit_id)
            df_track["DATE"] = date
            df_track["ID"] = commit_id
            # Concat
            df_tracks = pd.concat([df_tracks, df_track], axis=0)
        except Exception as e:
            print(f"Error on {date} - {commit_id}", e)
    print("Total notebooks tracked:", len(df_tracks))    
    return df_tracks

df_tracks = tracks_notebooks(df_pr)
df_tracks.head(1)
```

### Create trend dataframe


```python
def get_trend(df,
              date_col_name='DATE',
              date_val_name='ID',
              date_order='asc'):
    # Cleaning
    df = df.rename(columns={date_val_name: "VALUE"})
    
    df = df.groupby(date_col_name, as_index=False).agg({"VALUE": "count"})
    d = datetime.now().date()
    d2 = df.loc[df.index[0], date_col_name]
    idx = pd.date_range(d2, d, freq = "D")
    
    df.set_index(date_col_name, drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df[date_col_name] = pd.DatetimeIndex(df.index)
    df = df.reset_index(drop=True)
    for _, row in df.iterrows():
        if _ > 0:
            n_1 = df.loc[df.index[_-1], "VALUE"]
            n = df.loc[df.index[_], "VALUE"]
            if n == 0:
                df.loc[_, "VALUE"] = n_1
    df["GROUP"] = "Actual"
    
    # Create line to target
    value_target = 1000
    date_target = "2022-12-31"
    df_target = df[-1:].reset_index(drop=True)
    d = datetime.strptime(date_target, "%Y-%m-%d")
    d2 = df_target.loc[df_target.index[0], date_col_name]
    idx = pd.date_range(d2, d, freq = "D")
    df_target.set_index(date_col_name, drop=True, inplace=True)
    df_target.index = pd.DatetimeIndex(df_target.index)
    df_target = df_target.reindex(idx, fill_value=1000)
    df_target[date_col_name] = pd.DatetimeIndex(df_target.index)
    df_target = df_target.reset_index(drop=True)
    
    # Get target value
    to_go = value_target - df_target.loc[df_target.index[0], "VALUE"]
    var = to_go / (len(idx) - 1)
    for _, row in df_target.iterrows():
        if _ > 0:
            n_1 = df_target.loc[df_target.index[_-1], "VALUE"]
            df_target.loc[_, "VALUE"] = n_1 + var
    df_target["GROUP"] = "Forecast"

    # Concat data
    df = pd.concat([df, df_target]).reset_index(drop=True)
    df = df[[date_col_name, "GROUP", "VALUE"]]
    
    # Calc variation
    for idx, row in df.iterrows():
        if idx == 0:
            value_n1 = 0
        else:
            value_n1 = df.loc[df.index[idx-1], "VALUE"]
        df.loc[df.index[idx], "VALUE_COMP"] = value_n1
    df["VARV"] = df["VALUE"] - df["VALUE_COMP"]
    df["VARP"] = df["VARV"] / abs(df["VALUE_COMP"])
    
    # Prep data
    df["VALUE_D"] = df["VALUE"].map("{:,.0f}".format).str.replace(",", " ")
    df["VARV_D"] = df["VARV"].map("{:,.0f}".format).str.replace(",", " ")
    df.loc[df["VARV"] >= 0, "VARV_D"] = "+" + df["VARV_D"]
    df["VARP_D"] = df["VARP"].map("{:,.2%}".format).str.replace(",", " ")
    df.loc[df["VARP"] >= 0, "VARP_D"] = "+" + df["VARP_D"]
    
    df.loc[df["GROUP"] == "Forecast", "TEXT"] = ""
    df.loc[df["GROUP"] == "Actual", "TEXT"] = (df['VALUE_D'] + " notebooks as of " + df['DATE'].dt.strftime("%Y-%m-%d") +
                  " (" + df['VARV_D'] + " vs yesterday)")
    return df

df_trend = get_trend(df_tracks)
df_trend
```

## Output

### Plotting a line chart for notebook commits


```python
def create_linechart(df, label, group, value, text, title):
    # Init
    fig = go.Figure()
    
    # Create fig
    df1 = df[df[group] == "Actual"]
    last_text = df1.loc[df1.index[-1], text]
    fig.add_trace(
        go.Scatter(
            name="Actual",
            x=df1[label],
            y=df1[value],
            mode="lines",
            hovertext=df1[text],
            hoverinfo="text",
            line=dict(color="royalblue"),
        )
    )
    df2 = df[df[group] == "Forecast"]
    fig.add_trace(
        go.Scatter(
            name="Target",
            x=df2[label],
            y=df2[value],
            mode="lines",
            hovertext=df2[text],
            hoverinfo="text",
            line=dict(color="red", dash="dot"),
        )
    )
    fig.update_layout(
#         title=f"<b>Tracks of notebooks created since naas launch </b><br><span style='font-size: 13px;'>Total notebooks as of today: {notebooks} (+{var} vs yesterday)</span>",
        title=f"<b>{title}</b><br><span style='font-size: 13px;'>{last_text}</span>",
        title_font=dict(family="Verdana", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title='Date',
        xaxis_title_font=dict(family="Verdana", size=11, color="black"),
        yaxis_title='No. of notebooks',
        yaxis_title_font=dict(family="Verdana", size=11, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_trend, "DATE", "GROUP", "VALUE", "TEXT", chart_title)
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
naas.asset.add(image_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
