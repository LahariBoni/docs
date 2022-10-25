<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/Github_Create_Repo.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Github+Create+Repo:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #productivity #code #operations #snippet

**Author:** [Kanishk Pareek](https://in.linkedin.com/in/kanishkpareek/)

This notebook helps to Creating repo on github

## Input

### Importing library


```python
import requests
import json
```

### Variable


```python
# Enter your username here
user_name = "*********"
# Enter Your github token
github_token = "*************"
# Enter your repo name that you want to create
repo_name = "*************"
#Enter Your repo description
repo_description = "This is another repo"
```

## Model


```python
payload = {'name': repo_name, 'description': repo_description, 'auto_init': 'true'}
repo_request = requests.post('https://api.github.com/' + 'user/repos', auth=(user_name,github_token), data=json.dumps(payload))
```

## Output


```python
if repo_request.status_code == 422:
    print("Github repo already exists try wih other name.")
elif repo_request.status_code == 201:
    print("Github repo has created successfully.")
elif repo_request.status_code == 401:
    print("You are unauthorized user for this action.") 
```


```python

```
