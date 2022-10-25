<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Get_all_files_from_directory.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Get+all+files+from+directory:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #naas #glob #pprint #snippet

**Author:** [Ayoub Berdeddouch](https://www.linkedin.com/ayoub-berdeddouch)

**Description:** This notebook gives you the ability to get all files from a directory even in a sub-directory.

## Input

### Import libraries


```python
import glob
from os import path
```

### Setup Variables


```python
dir_path = "" #If dir path is "", it will returned files from current folder
```

## Model

### Get all files


```python
files = glob.glob(path.join(dir_path, "**"), recursive=True)
```

## Output

### Dislay result


```python
print(f"Total files and folders fetched:", len(files))
sorted(files)
```
