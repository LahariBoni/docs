<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Download_Content_Engine.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Download+Content+Engine:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #automation #linkedin #youtube #twitter #snapchat #instagram #facebook #tiktok #dataproduct

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook download the [Content Engine](https://github.com/jupyter-naas/naas-content-engine) data product to your naas.

## Input

### Import libraries


```python
from IPython.display import display
from ipywidgets import widgets
import naas
from os import path
```

### Define your project name


```python
PROJECT_NAME = "Naas Content Engine"
```

## Model

### Function


```python
# Setup folder path
GIT_REPO_PATH = path.join("/", "home", "ftp", PROJECT_NAME)

# Download repo inside folder
!git clone "https://github.com/jupyter-naas/naas-content-engine.git" "$GIT_REPO_PATH"
```

## Output

#### Download project on your naas


```python
print(f"✅ The project has been downloaded to the following folder: {GIT_REPO_PATH}.")
```


```python

```
