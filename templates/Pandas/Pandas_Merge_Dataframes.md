<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pandas/Pandas_Merge_Dataframes.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pandas+-+Merge+Dataframes:+Error+short+description">🚨 Bug report</a>

**Tags:** #pandas #python #merging #merge #dataframes #consolidate #operations #snippet #dataframe

**Author:** [Oketunji Oludolapo](https://www.linkedin.com/in/oludolapo-oketunji/)

This notebook will help you understand how to use the pandas merge function. It explains how to merge two datasets together and consolidate multiple dataset into one.

## Input

### Import Library


```python
import pandas as pd
import numpy as np
```

### Create dataframes to be merged

#### Dataframe 1


```python
# Creating values to be used as datasets
dict1 = {
    "student_id": [1,2,3,4,5,6,7,8,9,10],
    "student_name": ["Peter","Dolly","Maggie","David","Isabelle","Harry","Akin","Abbey","Victoria","Sam"],
    "student_course": np.random.choice(["Biology","Physics","Chemistry"], size=10)
}
```


```python
# Create dataframe
df_1 = pd.DataFrame(dict1)
df_1
```

#### Dataframe 2


```python
# Creating values to be used as datasets
dict2 = {
    "student_id": np.random.choice([1,2,3,4,5,6,7,8,9,10], size=100),
    "student_grade": np.random.choice(["A","B","C","D","E","F"], size=100),
    "professors": np.random.choice(["Mark Levinson","Angela Marge","Bonnie James","Klaus Michealson"], size=100),
}
```


```python
# Create dataframe
df_2 = pd.DataFrame(dict2)  # OR Data2=pd.read_csv(filepath)
df_2
```

## Model
pd.merge: acts like an SQL inner join and joins based on similar columns or index unless specified to join differently<br />

### Merging dataframes with same values with same column names
Using pd.merge(left, right) acts like sql inner join and only joins on the common column they have.<br>
It tries finding everything from the right and append to left 'student_id' is common to both so it has been merged into one and included all the other df_2 columns to df_1 table.<br>


```python
df = pd.merge(df_1, df_2)
```

## Output

### Display result


```python
df
```

## Other options

### Specifiying the comon column using parameters "on"


```python
df = pd.merge(df_1, df_2, on="student_id")
df
```

### Specifying what kind of Joins you want since merging does inner joins by default

- "inner" > Inner Join: INCLUDING ROWS OF FIRST AND SECOND ONLY IF THE VALUE IS THE SAME IN BOTH DATAFRAMES<br />
- "outer" > Outer Join: IT JOINS ALL THE ROWS OF FIRST AND SECOND DATAFRAMES TOGETHER AND CREATE NaN VALUE IF A ROW DOESN'T HAVE A VALUE AFTER JOINING<br />
- "left" > Left Join: INCLUDES ALL THE ROWS IN THE FIRST DATAFRAME AND ADDS THE COLUMNS OF SECOND DATAFRAME BUT IT WON'T INCLUDE THE ROWS OF THE SECOND DATAFRAME IF IT'S NOT THE SAME WITH THE FIRST<br />
- "right" > Right Join: INCLUDES ALL THE ROWS OF SECOND DATAFRAME AND THE COLUMNS OF THE FIRST DATAFRAME BUT WON'T INCLUDE THE ROWS OF THE FIRST DATAFRAME IF IT'S NOT SIMILAR TO THE SECOND DATAFRAME


```python
df = pd.merge(df_1, df_2, on="student_id", how='left')
df
```

### Merging dataframes with same values but different column names
We add two more parameters :<br>
- Left_on means merge using this column name<br>
- Right_on means merge using this column name<br>
i.e merge both id and student_id together<br>
since they don't have same name, they will create different columns on the new table


```python
df_1 = df_1.rename(columns={"student_id": "id"}) # Renamed student_id to id so as to give this example
df_1
```


```python
df = pd.merge(df_1, df_2, left_on="id", right_on="student_id")
df
```

### Merging with the index of the first dataframe


```python
df_1.set_index("id") # this will make id the new index for df_1
```


```python
df = pd.merge(df_1, df_2, left_index=True, right_on="student_id")#the new index will be from index of df_2 where they joined
df
```

### Merging both table on their index i.e two indexes


```python
df_2.set_index("student_id") # making student_id the index of Data2
```


```python
df = pd.merge(df_1, df_2, left_index=True, right_index=True) # new index will be from the left index unlike when joining only one index
df
```
