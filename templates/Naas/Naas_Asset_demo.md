<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Asset_demo.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Asset+demo:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #asset #snippet #operations

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

Read the doc on https://naas.gitbook.io/naas/features/asset

Build notebooks that can communicate with third parties applications and run remotely.

## Input

### Import library


```python
import naas 
```

### Setup Naas


```python
# Add the path of the document you want to share (don't forget to put the extension)
PATH = "PATH_OF_YOUR_DOCUMENT.png"
```

## Model

### Add your asset


```python
url = naas.asset.add(PATH)

# To delete your asset, uncomment the line below and execute this cell
# naas.asset.delete(PATH)
```

## Output

### Display result


```python
url
```
