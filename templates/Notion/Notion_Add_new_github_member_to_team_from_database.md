<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Add_new_github_member_to_team_from_database.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Add+new+github+member+to+team+from+database:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #teams #automation #notion #operations

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)


## Input

### Import libraries


```python
import requests
from naas_drivers import github, notion
import naas
import pandas as pd
```

### Setup Notion

- [Get your Notion integration token](https://docs.naas.ai/drivers/notion)
- Share integration with your database


### Setup Slack



```python
# Enter Token API
NOTION_TOKEN = "secret_LnpbBqBP5qjXXXXXXXXXXXXXX"

# Enter Database id
DATABASE_ID = "https://www.notion.so/7483186a133a47ff9af4df3495?XXXXXXXXXXXXXX"

# Enter column name of the members from notion
unique_column_name = "COLUMN_NAME"
```

### Setup Github
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
```

### Setup Naas scheduler


```python
#Schedule the notebook to run every day at 09:00 hrs
naas.scheduler.add(cron="0 9 * * *")

# To delete your scheduler, uncomment the line below and execute the cell 
# naas.scheduler.delete()
```

## Model

### Get profiles from Notion database



```python
# df_notion = notion.connect(NOTION_TOKEN).database.query(DATABASE_ID, query={})
df_notion = notion.connect(NOTION_TOKEN).database.get(DATABASE_ID).df()
df_notion.head()
```

### Get team members from GitHub


```python
def get_team(team_url):
    # Init variables
    df = pd.DataFrame()
    
    # Get team id
    team_id = team_url.split("/teams/")[-1]
    
    # Get all teams
    df_teams = github.connect(GITHUB_TOKEN).teams.get_profiles(GITHUB_TEAMS_URL)
    
    # Filter on team
    df = df_teams[df_teams.SLUG == team_id]
    print(f"{len(df)} members in team: {team_id}")
    return df.reset_index(drop=True)
    
df_teams = get_team(GITHUB_TEAMS_URL)
df_teams.head(5)
```

### Get new members


```python
def get_new_members(df_notion, df_teams):
    # Init output
    new_members = []
    
    # Get members from notion
    notion_members = df_notion[unique_column_name].unique().tolist()

    # Get teams members
    teams_members_email, teams_members_login = [], []
    for mail, name in zip(df_teams['EMAIL'].to_list(), df_teams['LOGIN_NAME'].to_list()):
        if str(mail) != 'nan' and mail != None:
            teams_members_email.append(mail.lower())
        
        if str(name)!='nan' and name != None:
            teams_members_login.append(name.lower())
    
    teams_members_email,teams_members_login = list(set(teams_members_email)), list(set(teams_members_login))
    
    # Get new members
    for member in notion_members:
        member = str(member).split("https://github.com/")[-1].split("/")[0].lower()
        if (member not in teams_members_login and member not in teams_members_email):
            new_members = new_members + [member]
    return new_members

new_members = get_new_members(df_notion, df_teams)
new_members
```

### Add new members from database


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
        req_url = f"https://api.github.com/orgs/{team_org}/teams/{team_id}/memberships/{member}"
        headers = {'Authorization': f'token {GITHUB_TOKEN}',
                   "Accept": "application/vnd.github.v3+json"}
        res = requests.put(req_url, headers=headers)
        try:
            res.raise_for_status()
        except Exception as e:
            print(e)
        
        if res.status_code == 200:
            print(f'✅ Member {member} successfully invited to your team {team_id}')
            res_json = res.json()
            data.append(res_json)
            
    # Send result as dataframe
    df = pd.DataFrame(data)
    return df.reset_index(drop=True)

df_new = add_members(GITHUB_TEAMS_URL, new_members)
```

## Output


```python
df_new
```
