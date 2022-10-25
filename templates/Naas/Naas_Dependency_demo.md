<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Dependency_demo.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Dependency+demo:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #dependency #snippet #operations

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

Read the doc on https://naas.gitbook.io/naas/features/asset

## Input

### Import library


```python
import naas 
```

## Model

### Path of your document


```python
PATH = "PATH_OF_YOUR_DOCUMENT.png"
```

## Output

### Add the dependency


```python
naas.dependency.add("YOUR_DOCUMENT.png")
```
