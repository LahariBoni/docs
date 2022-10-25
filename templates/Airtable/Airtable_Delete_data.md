<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Airtable/Airtable_Delete_data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Airtable+-+Delete+data:+Error+short+description">🚨 Bug report</a>

**Tags:** #airtable #database #productivity #spreadsheet #naas_drivers #operations #snippet #text

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import library


```python
from naas_drivers import airtable
```

### Variables


```python
API_KEY = 'API_KEY'
BASE_KEY = 'BASE_KEY'
TABLE_NAME = 'TABLE_NAME'
```

## Model

### Delete data


```python
data = airtable.connect(API_KEY,
                        BASE_KEY,
                        TABLE_NAME).delete_by_field('Name', 'Tom')
```

## Output

### Display result


```python
data
```
