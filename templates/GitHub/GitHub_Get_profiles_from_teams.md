<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Get_profiles_from_teams.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Get+profiles+from+teams:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #team #operations #snippet #dataframe

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

This notebook enables you to get a dataframe of all the team members in a Github organization.

## Input

### Import libraries


```python
from naas_drivers import github
```

### Setup Github

**How to find your personal access token on Github?** 
- First we need to create a personal access token to get the details of our organization from here: https://github.com/settings/tokens
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API. 
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive. 
- You can find description of each scope in docs here (https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


### Variables


```python
TEAM_URL = "https://github.com/orgs/jupyter-naas/teams"

GITHUB_TOKEN = "ghp_PITkKHTBIbcXXXXXXXXXXXXXX"
```

## Model


```python
df_teams = github.connect(GITHUB_TOKEN).teams.get_profiles(TEAM_URL)
```

## Output


```python
print(f'Dataset size -> {df_teams.shape}')
df_teams.head(25)
```
