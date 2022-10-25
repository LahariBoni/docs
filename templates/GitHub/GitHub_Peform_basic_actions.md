<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Peform_basic_actions.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Peform+basic+actions:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #productivity #code #operations #snippet

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import library


```python
from git_lib import Git
```

## Model

### Configuration attribute description
- username : Git username
- password : Git password
- github_url : Git url to clone
- branch : Branch name on which perform action
- target_folder : Folder name of which clone,commit,push,pull,checkout has to be performed
- action : Action you wish to perform **(clone,commit,push,pull,checkout)**
- commit_message : Any message you wish to pass while commit

### Git configuration


```python
config = {
            'username':'< Your github username >',
            'password':'< Your github password >',
            'github_url':'< Github url >',
            'branch':'< Github branch name >',
            'target_folder':'< Folder name >',
            'action':'< Github action >',
            'commit_message':'< Your message for commit >'
         }
```

## Output

### Execute Github connector


```python
git_instance = Git(config)
```
