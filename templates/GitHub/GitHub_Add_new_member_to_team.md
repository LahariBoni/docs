<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Add_new_member_to_team.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Add+new+member+to+team:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #teams #snippet #operations #invitations

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)


This notebook will allow you to add new member to any team of your organization.

## Input

### Import libraries


```python
import requests
from naas_drivers import github
import pandas as pd
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

# GitHub teams url
GITHUB_TEAMS_URL = "ENTER_YOUR_GITHUB_TEAMS_URL_HERE" # EXAMPLE : "https://github.com/orgs/jupyter-naas/teams/opensource-contributors"

# New members to add : str or list of members
GITHUB_NEW_MEMBERS = "ENTER_YOUR_NEW_USERS_HERE" # EXAMPLE : "FlorentLvr" or ["FlorentLvr", "Dr0p42"]
```

## Model

### Add members to team


```python
def add_members(team_url, new_members):
    # Init output
    df = pd.DataFrame()
    
    # Get org id and team id
    team_id = team_url.split("/teams/")[-1].split("/")[0]
    team_org = team_url.split("https://github.com/orgs/")[-1].split("/")[0]
    
    # If a particular member already is present in the team,
    # then it does not create a copy of that member. No need to worry :)
    if isinstance(new_members, str):
        new_members = [new_members]
    
    # Add new members to team
    data = []
    for member in new_members:
        member = member.split("https://github.com/")[-1].split("/")[0]
        req_url = f"https://api.github.com/orgs/{team_org}/teams/{team_id}/memberships/{member}"
        headers = {'Authorization': f'token {GITHUB_TOKEN}',
                   "Accept": "application/vnd.github.v3+json"}
        res = requests.put(req_url, headers=headers)
        res.raise_for_status()
        
        if res.status_code == 200:
            print(f'✅ Member {member} successfully invited to your team {team_id}')
            res_json = res.json()
            data.append(res_json)
            
    # Send result as dataframe
    df = pd.DataFrame(data)
    return df.reset_index(drop=True)

df_new = add_members(GITHUB_TEAMS_URL, GITHUB_NEW_MEMBERS)
```

## Output

### New members added


```python
df_new
```
