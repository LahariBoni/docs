<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Credits_Get_Balance.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Credits+Get+Balance:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #credits #snippet #operations

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

## Input

### Import library


```python
from naas_drivers import naascredits
```

## Model

### Function


```python
balance = naascredits.connect().get_balance()
```

## Output

### Display result


```python
balance
```
