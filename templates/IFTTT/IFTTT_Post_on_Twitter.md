<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/IFTTT/IFTTT_Post_on_Twitter.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=IFTTT+-+Post+on+Twitter:+Error+short+description">🚨 Bug report</a>

**Tags:** #ifttt #nocode #snippet #marketing #twitter

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

See the tutorial video on Youtube: https://www.youtube.com/watch?v=FvAM1aS4GFM

## Input

### Import library


```python
from naas_drivers import ifttt
```

### Variables


```python
twitter_post = """Hello world, this is my first automated post 
                with @JupyterNaas @IFTTT driver🔥
                """
event = "naas_demo"
key = "ke4AigvXI5-EABaowdLt4fju1aOUxeMxSXQoN***"
data = { "value1": twitter_post }
```

## Model

### Connect to IFTTT


```python
result = ifttt.connect(key)
```

## Output

### Display result


```python
result = ifttt.send(event, data)
```
