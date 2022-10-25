<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter%20Notebooks/Jupyter_Notebooks_Read_file_json.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Jupyter+Notebooks+-+Read+file+json:+Error+short+description">🚨 Bug report</a>

**Tags:** #jupyternotebooks #naas #jupyter-notebooks #read #snippet #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
import json
from pprint import pprint
```

### Variables


```python
# Input
notebook_path = "../template.ipynb"
```

## Model

### Open file


```python
with open(notebook_path) as f:
    nb = json.load(f)
```

## Output

### Display result


```python
pprint(nb)
```
