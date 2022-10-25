<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter/Jupyter_Restart_server.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Jupyter+-+Restart+server:+Error+short+description">🚨 Bug report</a>

**Tags:** #jupyter #server #restart #snippet #operations #naas

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
from naas_drivers import jupyter
from os import environ
```

### Setup your Jupyter


```python
# Jupyter token stored in your env
JUPYTER_TOKEN = environ.get('JUPYTERHUB_API_TOKEN')

# Username => email address linked to your jupyter account
username = "wsr@naas.ai"
```

## Model

### Restart server


```python
jupyter.connect(JUPYTER_TOKEN).restart_user(username)
```

## Output

### Display result


```python

```
