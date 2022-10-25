<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Get_commits_ranking_from_repository.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Get+commits+ranking+from+repository:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #repos #commits #stats #naas_drivers #plotly #linechart #operations #analytics #dataframe #html

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

## Input


```python
import pandas as pd
import plotly.express as px
from naas_drivers import github
import naas
```

## Setup Github
**How to find your personal access token on Github?**

- First we need to create a personal access token to get the details of our organization from here: https://github.com/settings/tokens
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API.
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive.
- You can find description of each scope in docs here (https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


```python
# Github repository url
REPO_URL = "https://github.com/jupyter-naas/awesome-notebooks"

# Github token
GITHUB_TOKEN = "ghp_CEvqR7QauDbNLRiIiwAC1v4xxxxxxxxxxxxx"
```

## Model

### Get commits from repository url


```python
df_commits = github.connect(GITHUB_TOKEN).repos.get_commits(REPO_URL)
df_commits
```

## Output

### Get commits ranking by user


```python
def get_commits(df):
    # Exclude Github commits
    df = df[(df.COMMITTER_EMAIL.str[-10:] != "github.com")]
    
    # Groupby and count
    df = df.groupby(["AUTHOR_NAME"], as_index=False).agg({"ID": "count"})
    
    # Cleaning
    df = df.rename(columns={"ID": "NB_COMMITS"})
    return df.sort_values(by="NB_COMMITS", ascending=False).reset_index(drop=True)

df = get_commits(df_commits)
df
```

### Plot a bar chart of commit activity


```python
def create_barchart(df, repository):
    # Get repository
    repository = repository.split("/")[-1]
    
    # Sort df
    df = df.sort_values(by="NB_COMMITS")
    
    # Calc commits
    commits = df.NB_COMMITS.sum()
    
    # Create fig
    fig = px.bar(df,
                 y="AUTHOR_NAME",
                 x="NB_COMMITS",
                 orientation='h',
                 title=f"GitHub - {repository} : Commits by user <br><span style='font-size: 13px;'>Total commits: {commits}</span>",
                 text="NB_COMMITS",
                 labels={"AUTHOR_NAME": "Author",
                         "NB_COMMITS": "Nb commits"}
                 )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        font=dict(family="Arial", size=14, color="black"),
        paper_bgcolor="white",
        xaxis_title=None,
        xaxis_showticklabels=False,
        yaxis_title=None,
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_barchart(df, REPO_URL)
```

### Save and export html


```python
output_path = f"{REPO_URL.split('/')[-1]}_commits_ranking.html"
fig.write_html(output_path)
naas.asset.add(output_path, params={"inline": True})
```
