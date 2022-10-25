<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Canny/Canny_Github_issue_update.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Canny+-+Github+issue+update:+Error+short+description">🚨 Bug report</a>

**Tags:** #canny #product #operations #snippet #github

**Author:** [Martin Donadieu](https://www.linkedin.com/in/martindonadieu)

## Input

### Install package


```python
pip install PyGithub
```

### Import librairies


```python
import requests
import json
from github import Github
```

### Insert your accounts details


```python
# For Github 
gihub_personal_token = "YOUR_GITHUB_TOKEN"                        # Settings/Developer settings/Personal access tokens
github_repo = "optimusprime2021/api-tester"                       # Github repository name

# For Canny
canny_post_url = "https://canny.io/api/v1/posts/list"             # Canny post url
canny_apikey = "CANNY_API_KEY"                                    # Canny api key
```

## Model

### Get Canny posts Dataframe


```python
response = requests.get(canny_post_url)
data = {"apiKey":canny_apikey,"id":"","limit":"100"}
response = requests.post(canny_post_url,data)
post_details = response.json()
```

### Check connection status


```python
if response.status_code == 200:
    print("Successfully connected to Canny")
elif response.status_code == 404:
    print("Couldn't connect to Canny, Please check the credentials")
    exit()
```

### Generating dataframe


```python
import pandas as pd
pd.set_option('mode.chained_assignment', None)
dd = post_details['posts']
df = pd.DataFrame(columns = dd[0].keys()) 
for i in range(len(dd)):
    df = df.append(dd[i], ignore_index=True)
# df

board = []
category = []
tags = []
for i in range(len(df)):
    board.append(df['board'][i]['name'])
    if not df['category'][i]:
        category.append('Not assigned')
    else:
        category.append(df['category'][i]['name'])    
    if not df['tags'][i]:
        tags.append('Not assigned')
    else:
        tags.append(df['tags'][i][0]['name'])
        
        
df = df[['title','status','details','url']]
df['board'] = board
df['category'] = category
df['tags'] = tags
df = df[(df["tags"] == "Awesome-notebooks")]          # tag name
df
```


```python
## add url to dataframe
```

### Existing issue list


```python
issues = []
g = Github(gihub_personal_token)
repo = g.get_repo(github_repo)
open_issues = repo.get_issues(state='open')
for issue in open_issues:
    issues.append(issue.title)
```

## Output

### Push all issues


```python
repo = g.get_repo(github_repo)
for i in df.index:
    if df['title'][i] not in issues:
        repo.create_issue(title=df['title'][i], body=df['details'][i]+"\n canny url: "+df['url'][i])
```

### Close all issues


```python
# repo = g.get_repo(github_repo)
# open_issues = repo.get_issues(state='open')
# for issue in open_issues:
#     issue.edit(state='closed')
```
