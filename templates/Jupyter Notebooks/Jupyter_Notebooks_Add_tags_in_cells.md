<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter%20Notebooks/Jupyter_Notebooks_Add_tags_in_cells.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Jupyter+Notebooks+-+Add+tags+in+cells:+Error+short+description">🚨 Bug report</a>

**Tags:** #jupyternotebooks #jupyter #awesome-notebooks #tags #snippet #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
import os
import json
from pprint import pprint
```

### Setup your variables


```python
# Notebook path to add tags
notebook_path = "AWS/AWS_Upload_file_to_S3_bucket.ipynb"

# Tags to added
new_tags = [f"awesome-notebooks/{notebook_path}"]
```

## Model

### Add tags in cells


```python
def add_tags(notebook_path, new_tags):
    with open(notebook_path) as f:
        nb = json.load(f)
        
    new_cells = []
    cells = nb.get("cells")
    
    # Apply change
    for cell in cells:
        tags = cell.get('metadata').get('tags')
        if len(tags) == 0:
            tags = new_tags
        else:
            tags += new_tags
        cell["metadata"]["tags"] = tags
        new_cells.append(cell)
        
    # Save notebook
    nb_new = nb.copy()
    nb_new["cells"] = new_cells
    with open(notebook_path, 'w') as f:
        json.dump(nb_new, f)
    print(f"✔️ {notebook_path} saved in Naas.")
    return nb_new

nb = add_tags(notebook_path, new_tags)
```

## Output

### Display result


```python
pprint(nb)
```
