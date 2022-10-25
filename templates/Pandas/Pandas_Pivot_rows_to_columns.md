<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pandas/Pandas_Pivot_rows_to_columns.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pandas+-+Pivot+rows+to+columns:+Error+short+description">🚨 Bug report</a>

**Tags:** #pandas #pivot #snippet #operations #utils #data

**Author:** [Ismail Chihab](https://www.linkedin.com/in/ismail-chihab-4b0a04202/)

With this template, you can pivot a rows to columns.

## Input

### Import library


```python
import pandas as pd
```

### Setup Data


```python
data = {
    'LABEL' : ["Sales", "Sales", "Sales", "Gross Profit", "Gross Profit", "Gross Profit", "EBIT", "EBIT", "EBIT"],
    'DATE' : ['Jan', 'Feb', 'Mar', 'Jan', 'Feb', 'Mar', 'Jan', 'Feb', 'Mar'],
    'VALUE' : [0, 2, 3, 4, 5, 6, 7, 8, 9]
}
df = pd.DataFrame(data)
df
```

### Setup Variables


```python
# Name of the column you want to pivot
col_pivot = "DATE"

# Name of the column containing the values
col_value = "VALUE"

# List of columns not be pivot
cols_index = ["LABEL"]
```

## Model:


```python
def pivot_data(df_init, col_pivot, col_value, cols_index):
    # Drop duplicates
    df = df_init.copy()
    df = df.drop_duplicates()
    columns = df[col_pivot].unique().tolist()
    
    # Pivot 
    df = pd.pivot(df,
                  index=cols_index,
                  values=col_value,
                  columns=col_pivot)
    for col in cols_index:
        df.loc[:, col] = df.index.get_level_values(0)
    df = df.reset_index(drop=True)
    df = df.reindex(columns=cols_index+columns)
    return df
```

## Output

### Display result


```python
table = pivot_data(df, col_pivot, col_value, cols_index)
table
```
