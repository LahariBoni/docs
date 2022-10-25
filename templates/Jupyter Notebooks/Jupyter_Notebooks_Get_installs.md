<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter%20Notebooks/Jupyter_Notebooks_Get_installs.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Jupyter+Notebooks+-+Get+installs:+Error+short+description">🚨 Bug report</a>

**Tags:** #jupyternotebooks #naas #jupyter-notebooks #read #installs #snippet #operations #text

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

### Get module install in notebook


```python
def get_installs(notebook_path):
    with open(notebook_path) as f:
        nb = json.load(f)
    data = []
    
    cells = nb.get("cells")
    # Check each cells
    for cell in cells:
        cell_type = cell.get('cell_type')
        sources = cell.get('source')
        for source in sources:
            if cell_type == "code":
                if "pip install" in source:
                    install = source.split("pip install")[-1].replace("\n", "").strip()
                    data.append(install)
    if len(data) == 0:
        print("❎ No install found in notebook:", notebook_path)
    else:
        print(f"✅ {len(data)} install(s) found in notebook:", notebook_path)
    return data
```

## Output

### Display result


```python
installs = get_installs(notebook_path)
print(installs)
```


```python

```
