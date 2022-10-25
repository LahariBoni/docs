<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/SharePoint/SharePoint_List_folder.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=SharePoint+-+List+folder:+Error+short+description">🚨 Bug report</a>

**Tags:** #sharepoint #productivity #naas_drivers #operations #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook gives you the content of your folder in SharePoint.

## Input

### Import libraries


```python
from naas_drivers import sharepoint
import naas
```

### Setup SharePoint


```python
# SharePoint credentials
SP_ENDPOINT = naas.secret.get("SP_ENDPOINT") or "ENTER_YOUR_SP_ENDPOINT_HERE"
SP_USERNAME = naas.secret.get("SP_USERNAME") or "ENTER_YOUR_SP_USERNAME_HERE"
SP_PASSWORD = naas.secret.get("SP_PASSWORD") or "ENTER_YOUR_SP_PASSWORD_HERE"
SP_SITE = naas.secret.get("SP_SITE") or "ENTER_YOUR_SP_SITE_HERE"

# Folder path
FOLDER_PATH = "ENTER_YOUR_FOLDER_PATH_HERE"
```

## Model

### List folder


```python
folder = sharepoint.connect(SP_ENDPOINT, SP_USERNAME, SP_PASSWORD, SP_SITE).list_folder(FOLDER_PATH)
```

## Output

### Display result


```python
folder
```
