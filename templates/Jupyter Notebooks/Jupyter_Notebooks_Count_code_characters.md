<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter%20Notebooks/Jupyter_Notebooks_Count_code_characters.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a>

**Tags:** #jupyternotebooks #naas #jupyter-notebooks #read #codecharacters #snippet #operations #text

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
def count_characters(notebook_path):
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
                    char = source.replace(" ", "")
                    data += len(char)
    if data == 0:
        print("❎ No character of code wrote in notebook:", notebook_path)
    else:
        print(f"✅ {data} character(s) of code wrote in notebook:", notebook_path)
    return data
```

## Output

### Display result


```python
no_characters = count_characters(notebook_path)
no_characters
```
