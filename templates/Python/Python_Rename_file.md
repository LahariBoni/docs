<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Rename_file.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Rename+file:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #consolidate #files #productivity #snippet #operations #excel

**Author:** [Divakar](https://www.linkedin.com/in/divakar-r-9b34b86b/)

The objective of this notebook is to rename files. 

## Input 

### Import library
Import the necessary libraries: os


```python
import os
```

### Setup variables


```python
input_file_path = "test1.ppt"
output_file_path = "test2.ppt"
```

## Model

### Rename file using os


```python
os.rename(input_file_path, output_file_path)
```

## Output

### Display result


```python
print(f"File {input_file_path} successfully renamed: {output_file_path}!")
```
