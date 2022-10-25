<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Boursorama/Boursorama_Get_CDS.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Boursorama+-+Get+CDS:+Error+short+description">🚨 Bug report</a>

**Tags:** #boursorama #finance #snippet #dataframe

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import library


```python
import pandas as pd
```

### Variables


```python
URL = "https://www.boursorama.com/bourse/taux/cds/"
```

## Model

### Get the CDS from URL 


```python
dfs = pd.read_html(URL)
df = dfs[0]
```

## Output

### Display result


```python
df
```

### Save as csv


```python
df.to_excel(r'Get_CDS.csv', index = False)
```
