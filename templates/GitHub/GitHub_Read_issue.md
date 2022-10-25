<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Read_issue.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Read+issue:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #productivity #code #operations #snippet

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import Library


```python
from github import Github
```

### Enter repository path and token


```python
repo_name = "**********"                # Repository path
git_key = "**********"                  # Settings/Developer settings
```

## Model

### Establishing connection


```python
g = Github(git_key)   
```

### See issue list


```python
import pandas as pd
repo = g.get_repo(repo_name)
open_issues = repo.get_issues(state='open')
title = []
number = []
for issue in open_issues:
    title.append(issue.title)
    number.append(issue.number)
data = {'Title':title,'Description':title, 'ID':number} 
df = pd.DataFrame(data)
df
```

## Output

### Display issue list


```python
df
```
