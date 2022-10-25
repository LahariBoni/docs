<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Get_pull_requests_from_repository.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Get+pull+requests+from+repository:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #repos #pulls #PR #operations #analytics #plotly #dataframe

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

The objective of this notebook is to maintain a track of open pull requests that have been pending a review since more than 7 days within a repository

## Input

### Imports


```python
from naas_drivers import github
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

## Model


```python
df_pulls = github.connect(GITHUB_TOKEN).repos.get_pulls(REPO_URL)
```

## Output

### Display result


```python
print("Opened PR:", len(df_pulls))
df_pulls
```
