<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Get_Transactions.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Get+Transactions:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #credits #snippet #operations

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

## Input

### Import library


```python
from naas_drivers import naascredits
import pandas as pd
```

### Variables


```python
page_size = "100000"
```

## Model

### Function


```python
df = pd.DataFrame(naascredits.connect().transactions.get(page_size=page_size))
df
```

## Output

### Display result


```python
df.to_csv('billing.csv')
```
