<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Remove_member_from_team.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Remove+member+from+team:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #teams #snippet #operations

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)


## Input

### Import libraries


```python
import requests
from naas_drivers import github
```

## Setup Github
**How to find your personal access token on Github?**

- First we need to create a personal access token to get the details of our organization from [here](https://github.com/settings/tokens)
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API.
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive.
- You can find description of each scope in docs [here](https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


```python
# GitHub token
GITHUB_TOKEN = "ENTER_YOUR_GITHUB_TOKEN_HERE" # EXAMPLE : "ghp_fUYP0Z5i29AG4ggX8owctGnHU**********"

# GitHub teams url
GITHUB_TEAMS_URL = "ENTER_YOUR_GITHUB_TEAMS_URL_HERE" # EXAMPLE : "https://github.com/orgs/jupyter-naas/teams/opensource-contributors"

# members to remove : str or list of members
GITHUB_MEMBERS = "ENTER_YOUR_USERS_TO_BE_DELETED_HERE" # EXAMPLE : "FlorentLvr" or ["FlorentLvr", "Dr0p42"]
```

## Model

### Remove members to team


```python
def remove_members(team_url, members):
    # Init output
    members = []
    
    # Get org id and team id
    team_id = team_url.split("/teams/")[-1].split("/")[0]
    team_org = team_url.split("https://github.com/orgs/")[-1].split("/")[0]
    
    # Headers
    headers = {'Authorization': f'token {GITHUB_TOKEN}',
                   "Accept": "application/vnd.github.v3+json"}
    
    if isinstance(members, str):
        members = [members]
    
    # Remove members from team
    for member in members:
        member = member.split("https://github.com/")[-1].split("/")[0]
        req_url = f"https://api.github.com/orgs/{team_org}/teams/{team_id}/memberships/{member}"
        res = requests.delete(req_url, headers=headers)
        res.raise_for_status()
        if res.status_code == 204:
            print(f'✅ Member {member} successfully removed from your team {team_id}')
            data = {
                "MEMBER": member,
                "TEAM": team_id,
                "STATUS": "Removed"
            }
            members.appen(member)
    return pd.DataFrame(members)
    

df_remove = remove_members(GITHUB_TEAMS_URL, GITHUB_MEMBERS)
```

## Output

### Member(s) removed


```python
df_remove
```

The members are finally removed from the github teams :) !!
