<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter%20Notebooks/Jupyter_Notebooks_Count_code_lines.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Jupyter+Notebooks+-+Count+code+lines:+Error+short+description">🚨 Bug report</a>

**Tags:** #jupyternotebooks #naas #jupyter-notebooks #read #codelines #snippet #operations #text

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
import json
```

### Variables


```python
# Input
notebook_path = "../template.ipynb"
```

## Model

### Get module libraries in notebook


```python
def count_codes(notebook_path):
    with open(notebook_path) as f:
        nb = json.load(f)
    data = 0
    
    cells = nb.get("cells")
    # Check each cells
    for cell in cells:
        cell_type = cell.get('cell_type')
        sources = cell.get('source')
        for source in sources:
            if cell_type == "code":
                if not source.startswith('\n') and not source.startswith('#'):
                    data += 1
    if data == 0:
        print("❎ No line of code wrote in notebook:", notebook_path)
    else:
        print(f"✅ {data} line(s) of code wrote in notebook:", notebook_path)
    return data
```

## Output

### Display result


```python
no_lines = count_codes(notebook_path)
no_lines
```


```python

```
