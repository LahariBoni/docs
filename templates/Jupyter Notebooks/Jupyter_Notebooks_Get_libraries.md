<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter%20Notebooks/Jupyter_Notebooks_Get_libraries.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Jupyter+Notebooks+-+Get+libraries:+Error+short+description">🚨 Bug report</a>

**Tags:** #jupyternotebooks #naas #jupyter-notebooks #read #libraries #snippet #operations #text

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

### Get module libraries in notebook


```python
def get_libraries(notebook_path):
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
                if "from" in source and "import" in source:
                    lib = source.replace("\n", "").split("from")[-1].split("import")[0].strip()
                    module = source.replace("\n", "").split("import")[-1].split(" as ")[0].strip()
                    data.append(f"{lib}.{module}")
                if "from" not in source and "import" in source:
                    library = source.replace("\n", "").split("import")[-1].split(" as ")[0].strip()
                    data.append(library)
    if len(data) == 0:
        print("❎ No library found in notebook:", notebook_path)
    else:
        print(f"✅ {len(data)} librarie(s) found in notebook:", notebook_path)
    return data
```

## Output

### Display result


```python
libraries = get_libraries(notebook_path)
print(libraries)
```


```python

```
