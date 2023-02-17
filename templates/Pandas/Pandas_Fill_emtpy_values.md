<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pandas/Pandas_Fill_emtpy_values.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pandas+-+Fill+emtpy+values:+Error+short+description">Bug report</a>

**Tags:** #pandas #snippet #datacleaning #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

**Description:** This notebook fill empty values in dataframe columns.

## Input

### Import libraries


```python
import pandas as pd
import numpy as np
```

### Setup DataFrame


```python
# create DataFrame
df = pd.DataFrame(
    {
        "team": ["A", "A", "A", "B", "B", np.NaN],
        "points": [11, 7, 8, 10, np.NaN, 13],
        "assists": [5, 7, 7, 9, 12, np.NaN],
        "rebounds": [np.NaN, 8, 10, 6, 6, np.NaN],
    }
)
df
```

### Define data to fill if empty values in columns


```python
# dict of columns with value to fill
to_fillna = {"team": "Unknown team", "points": 0, "assists": 0, "rebounds": 0}
```

## Model

### Fill empty values in dataframe


```python
df = df.fillna(to_fillna)
```

## Output

### View new DataFrame


```python
df
```