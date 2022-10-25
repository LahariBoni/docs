<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Looping_Over_Dataframe.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Looping+Over+Dataframe:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #pandas #python #loops #dataframes #forloop #loop #snippet #operations

**Author:** [Oketunji Oludolapo](https://www.linkedin.com/in/oludolapo-oketunji/)

This notebook will help you in looping over your tables and getting concise information

## Input

### Import Library


```python
import pandas as pd
import numpy as np
```

## Model
Loopring over dataframes can be a a lifesaver when there are lots of columns and we want to view our data at a go. This is an advntage of for loop in a dataframe.

### Create Sample Dataframe 


```python
dict1 = {
    "student_id": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    "student_name": ["Peter", "Dolly", "Maggie", "David", "Isabelle", "Harry", "Akin", "Abbey", "Victoria", "Sam"],
    "student_course": np.random.choice(["Biology", "Physics", "Chemistry"], size=10)
}
```


```python
data = pd.DataFrame(dict1)
```


```python
data
```

## Output

### Looping over the data to get the column name


```python
for column in data:
    print(column)
```

### Looping over the data to view the columns and their values sequentially


```python
for k, v in data.iteritems():
    print(k)
    print(v)
```

### Looping over dataframes to get the informtion about a row with respect to its columns


```python
for k, v in data.iterrows():
    print(k)
    print(v)
```

### Looping over datafrmes to view the data per row as a tuple with the column values


```python
for row in data.itertuples():
    print(row)
```
