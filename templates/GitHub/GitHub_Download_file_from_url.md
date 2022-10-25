<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Download_file_from_url.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Download+file+from+url:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #productivity #code #operations #snippet #dataframe

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import needed library


```python
import requests
import naas
import uuid
```

## Model

### Default Github file for testing purpose


```python
target = "https://github.com/jupyter-naas/awesome-notebooks/blob/master/Plotly/Create%20Candlestick%20chart.ipynb"
```

### Convert url to downloadable one


```python
# https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dataviz/Plotly/Create%20Candlestick%20chart.ipynb
raw_target = target.replace("https://github.com/", "https://raw.githubusercontent.com/")
raw_target = raw_target.replace("/blob/", "/")
print(raw_target)
```

## Output

### Dowload file locally


```python
import urllib.parse
r = requests.get(raw_target)
uid = uuid.uuid4().hex

file_name = raw_target.split('/')[-1]
file_name = urllib.parse.unquote(file_name)

with open(file_name, 'wb') as f:
    f.write(r.content)
```
