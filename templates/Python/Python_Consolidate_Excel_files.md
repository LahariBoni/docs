<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Consolidate_Excel_files.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Consolidate+Excel+files:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #consolidate #files #productivity #snippet #operations #excel

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

The objective of this notebook is to consolidate multiple Excel files (.xlsx) into one. 

## Input 

### Import library
Import the necessary libraries: os and pandas 


```python
import os
import pandas as pd
```

### Variables


```python
# Output
excel_output = 'concatenate.xlsx'
```

## Model
Use a for loop to 
- List all the files in the current directory with os.listdir().
- Filter files with the .endswith(‘.xlsx’) method.
- Make sure the files will be stored into a list called my_list and then combined with pd.concat()

Then
- Return a dataframe and name it df_concat. 


```python
files = os.listdir()
my_list = []
for file in files:
    if file.endswith('.xlsx'):
        df = pd.read_excel(file)
        my_list.append(df)

df_concat = pd.concat(my_list, axis=0)
```

## Output
Export your dataframe to an Excel file.


```python
df_concat.to_excel(excel_output, index=False)
```
