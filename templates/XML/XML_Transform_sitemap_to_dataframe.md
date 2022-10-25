<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/XML/XML_Transform_sitemap_to_dataframe.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=XML+-+Transform+sitemap+to+dataframe:+Error+short+description">🚨 Bug report</a>

**Tags:** #xml #file #tool #operations #automation #dataframe

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import library


```python
import naas
import json 
try:
    import xmltodict
except:
    !pip install xmltodict
    import xmltodict
import pandas as pd
import requests
```

### Choose the website you want


```python
website = "https://zapier.com"
```

## Model

### Get your Dataframe


```python
def sitemap_to_df(url):
    df = None
    key = "urlset.url.url"
    r = requests.get(f'{url}/sitemap.xml')
    data_dict = xmltodict.parse(r.content) 
    if key and len(key.split('.')) > 0:
        keys = key.split('.')
        keys.reverse()
        data = data_dict.get(keys.pop())
        while(len(keys) > 1):
            data = data.get(keys.pop())
        df = pd.DataFrame.from_dict(data=data)
    elif key and data_dict.get(key):
        df = pd.DataFrame.from_dict(data=data_dict.get(key))
    else:
        df = pd.DataFrame.from_dict(data=data_dict)
    return df
```


```python
df = sitemap_to_df(website)
```

## Output

### Display result


```python
df
```

### Set the timezone


```python
naas.get_remote_timezone()
```


```python
naas.set_remote_timezone("Europe/Lisbon")
```
