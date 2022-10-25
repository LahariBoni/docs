<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Sheets/Google_Sheets_Add_new_github_member_to_team_from_spreadsheet.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Sheets+-+Add+new+github+member+to+team+from+spreadsheet:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #teams #automation #googlesheets #operations

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)


## Input

### Import libraries


```python
import requests
from naas_drivers import github, gsheet
import naas
import pandas as pd
```

### Setup Google Sheet
- Share your Google Sheet with our service account : 🔗 naas-share@naas-gsheets.iam.gserviceaccount.com


```python
# Enter spreadsheet URL and sheet name
SPREADSHEET_URL = "ENTER_YOUR_SPREADSHEET_URL_HERE" # EXAMPLE : "https://docs.google.com/spreadsheets/d/**********" 
SHEET_NAME = "ENTER_YOUR_SHEET_NAME_HERE" # EXAMPLE : "**********" 

# GitHub profile column name in Google Sheets
col_unique_gsheet = 'ENTER_YOUR_COLUMN_NAME_HERE' # EXAMPLE : "**********" 
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

### Get Google Sheets data


```python
# Connect with Gsheet and get all data in a dataframe
df_gsheet = gsheet.connect(SPREADSHEET_URL).get(sheet_name=SHEET_NAME)
df_gsheet.head(5)
```

### Get team from GitHub


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
def get_new_members(df_gsheet, df_teams):
    # Init output
    new_members = []
    
    # Get gsheet members
    gsheet_members = df_gsheet[col_unique_gsheet].unique().tolist()
    
    # Get teams members
    teams_members_login = df_teams["LOGIN_NAME"].str.lower().unique().tolist()
    print(teams_members_login)
    teams_members_email = df_teams["EMAIL"].str.lower().unique().tolist()
    
    # Get new members
    for member in gsheet_members:
        member = str(member).split("https://github.com/")[-1].split("/")[0].lower()
        if (member not in teams_members_login and member not in teams_members_email):
            new_members = new_members + [member]
    return new_members

new_members = get_new_members(df_gsheet, df_teams)
new_members
```

### Add members from spreadsheet


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

### Display new members added


```python
df_new
```
