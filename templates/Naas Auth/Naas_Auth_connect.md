<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas%20Auth/Naas_Auth_connect.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+Auth+-+Connect:+Error+short+description">🚨 Bug report</a>

**Tags:** #naasauth #naas #auth #operations #snippet

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou/)

## Input

### Import library


```python
from naas_drivers import naasauth
```

## Model

### Function

When calling connect() you can either pass a token generated [here](https://app.naas.ai/hub/token) or pass nothing. If you choose to pass nothing and that you are running this notebook on [app.naas.ai](https://app.naas.ai) it will use your jupyter token automatically.


```python
#naasauth.connect(token="yourjupyterhubtoken")
#naasauth.connect("yourjupyterhubtoken")
naasauth.connect()
```

## Output

### Display result


```python
naasauth.access_token
```


```python
naasauth.headers
```
