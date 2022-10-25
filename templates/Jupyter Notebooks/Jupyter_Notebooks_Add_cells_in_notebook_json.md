<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Jupyter%20Notebooks/Jupyter_Notebooks_Add_cells_in_notebook_json.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Jupyter+Notebooks+-+Add+cells+in+notebook+json:+Error+short+description">🚨 Bug report</a>

**Tags:** #jupyternotebooks #naas #jupyter-notebooks #snippet #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
import json
from pprint import pprint
```

### Variables


```python
notebook_path = "Jupyter_Notebooks_Get_libraries.ipynb"
```

## Model

### Add new cells "Author" and "Tags" in awesome-notebooks if does not exist


```python
def add_cells(notebook_path):
    notebook_path
    with open(notebook_path) as f:
        nb = json.load(f)
    root = notebook_path.split("/")[0]
    check_logo = False
    check_title = False
    check_download = False
    check_tags = False
    check_author = False
    check_input = False
    check_model = False 
    check_output = False
    no_markdown = 0
    
    cells = nb.get("cells")
    # Check each cells
    for cell in cells:
        cell_type = cell.get('cell_type')
        sources = cell.get('source')    
        if cell_type == "markdown":
            for source in sources:
                if source.startswith("<img") and no_markdown == 0:
                    check_logo = True
                if source.startswith("# ") and root in source:
                    check_title = True
                if "https://app.naas.ai/user-redirect/naas/downloader" in source:
                    check_download = True
                if "**Tags:**" in source:
                    check_tags = True
                    tags = source[9:].strip()
                if "**Author:**" in source:
                    check_author = True
                    author = source[11:].split("[")[-1].split("]")[0]
                if source.startswith("## Input"):
                    check_input = True
                if source.startswith("## Model"):
                    check_model = True
                if source.startswith("## Output"):
                    check_output = True
                
    # Check
    add_logo = False
    add_author = False
    if not check_logo:
        print("Logo to be added")
        add_logo = True
    if not check_author and check_tags:
        print("Author to be added below tags")
        add_author = True
        
    # Apply change
    new_cells = []
    if add_logo:
        cell_logo = {'cell_type': 'markdown',
                     'id': 'naas-logo',
                     'metadata': {'papermill': {}, 'tags': ["naas"]},
                     'source': '<img width="10%" alt="Naas" src="https://landen.imgix.net/jtci2pxwjczr/assets/5ice39g4.png?w=160"/>'}
        new_cells.append(cell_logo)
    for cell in cells:
        new_cells.append(cell)
        cell_type = cell.get('cell_type')
        source = cell.get('source')

        if cell_type == "markdown":
            if "**Tags:**" in source and add_author:
                cell_author = {'cell_type': 'markdown',
                               'id': 'naas-author',
                               'metadata': {'papermill': {}, 'tags': ["naas"]},
                               'source': '**Author:** [Unknown](https://www.linkedin.com/company/naas-ai/)'}
                new_cells.append(cell_author)
    if add_logo or add_author:
        nb_new = nb.copy()
        nb_new["cells"] = new_cells
        with open(notebook_path, 'w') as f:
            json.dump(nb_new, f)
        print(f"{notebook_path} saved in Naas.")
        return nb_new
    else:
        print("Nothing to be added in notebooks !")
```

## Output

### Display result


```python
nb = add_cells(notebook_path)
pprint(nb)
```


```python

```
