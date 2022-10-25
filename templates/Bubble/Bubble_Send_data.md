<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Bubble/Bubble_Send_data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Bubble+-+Send+data:+Error+short+description">🚨 Bug report</a>

**Tags:** #bubble #naas_drivers #operations #snippet

**Author:** [Martin Donadieu](https://www.linkedin.com/in/martindonadieu/)

## Input

### Import library


```python
from naas_drivers import bubble
```

### Variables


```python
url = "https://appname.bubbleapps.io/api/1.1/wf/endpoint_name"
data = {"first_name":"Bryan",
        "last_name":"Helmig",
        "age": 27 }
```

## Model

### Trigger workflow


```python
def send_data(url, data):
    bubble.send(url, data)
```

## Output

### Send result


```python
result = send_data(url, data)
```
