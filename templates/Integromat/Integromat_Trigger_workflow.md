<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Integromat/Integromat_Trigger_workflow.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Integromat+-+Trigger+workflow:+Error+short+description">🚨 Bug report</a>

**Tags:** #integromat #snippet #operations

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import libraries


```python
from naas_drivers import integromat
```

### Setup Integromat


```python
URL = "https://hook.integromat.com/7edtlwmn8foer0r9i9ainjvsz3vxmwos"

DATA = {"first_name": "Bryan",
        "last_name":"Helmig", "age": 27 }
```

## Model

### Send data


```python
result = integromat.connect(URL).send(DATA)
```

## Output

### Display result


```python
result
```
