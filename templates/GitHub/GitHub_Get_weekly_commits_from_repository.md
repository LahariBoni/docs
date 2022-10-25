<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Get_weekly_commits_from_repository.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Get+weekly+commits+from+repository:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #repos #commits #stats #naas_drivers #plotly #linechart #operations #analytics #dataframe #html

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

## Input


```python
import pandas as pd
import plotly.express as px
from naas_drivers import github
import naas
```

### Setup Github
**How to find your personal access token on Github?**

- First we need to create a personal access token to get the details of our organization from here: https://github.com/settings/tokens
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API.
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive.
- You can find description of each scope in docs here (https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


```python
# GitHub token
GITHUB_TOKEN = "ENTER_YOUR_GITHUB_TOKEN_HERE" # EXAMPLE : "ghp_fUYP0Z5i29AG4ggX8owctGnHU**********" 

# Github repo on which we want to create issues.
REPO_URL = "ENTER_YOUR_REPO_URL_HERE" # EXAMPLE : "https://github.com/jupyter-naas/awesome-notebooks/"
```

## Model

### Get commits from repository url


```python
df_commits = github.connect(GITHUB_TOKEN).repos.get_commits(REPO_URL)
df_commits
```

### Get weekly commits


```python
def get_weekly_commits(df):
    # Exclude Github commits
    df = df[(df.COMMITTER_EMAIL.str[-10:] != "github.com")]
    
    # Groupby and count
    df = df.groupby(pd.Grouper(freq='W', key='AUTHOR_DATE')).agg({"ID": "count"}).reset_index()
    df["WEEKS"] = df["AUTHOR_DATE"].dt.strftime("W%U-%Y")
    
    # Cleaning
    df = df.rename(columns={"ID": "NB_COMMITS"})
    return df

df_weekly = get_weekly_commits(df_commits)
df_weekly
```

### Plot a bar chart of weekly commit activity


```python
def create_barchart(df, repository):
    # Get repository
    repository = repository.split("/")[-1]
    
    # Calc commits
    commits = df.NB_COMMITS.sum()
    
    # Create fig
    fig = px.bar(df,
           title=f"GitHub - {repository} : Weekly user commits <br><span style='font-size: 13px;'>Total commits: {commits}</span>",
           x="WEEKS",
           y="NB_COMMITS",
           labels={
               'WEEKS':'Weeks committed',
               'NB_COMMITS':"Nb. commits"
          })
    fig.update_traces(marker_color='black')
    fig.update_layout(
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        font=dict(family="Arial", size=14, color="black"),
        paper_bgcolor="white",
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_barchart(df_weekly, REPO_URL)
```

## Output

### Save and export html


```python
output_path = f"{REPO_URL.split('/')[-1]}_weekly_commits.html"
fig.write_html(output_path)
naas.asset.add(output_path, params={"inline": True})
```


```python

```
