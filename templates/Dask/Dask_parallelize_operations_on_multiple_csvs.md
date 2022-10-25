<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Dask/Dask_parallelize_operations_on_multiple_csvs.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Dask+-+Parallelize+operations+on+multiple+csvs:+Error+short+description">🚨 Bug report</a>

**Tags:** #csv #pandas #snippet #read #dataframe #parallel #parallelize #dask #operations

**Author:** [Minura Punchihewa](https://www.linkedin.com/in/minurapunchihewa/)

## Input

### Imports


```python
import os
```

### Import Graphviz (install if not present)


```python
try:
    import graphviz
except:
    !pip install --user graphviz
    import graphviz
```

### Import Dask (install if not present)


```python
try:
    import dask.dataframe as dd
except:
    !python -m pip install "dask[complete]"
    import dask.dataframe as dd
```

### Variable


```python
folder_path = "nycflights"

%env FOLDER_PATH=$folder_path
```

### Download dataset if it does not exists


```bash
%%bash

[[ -f "$FOLDER_PATH/nycflights.csv" ]] || (mkdir -p $FOLDER_PATH && wget -O $FOLDER_PATH/nycflights.csv  https://github.com/vaibhavwalvekar/NYC-Flights-2013-Dataset-Analysis/raw/master/flights.csv )
```

## Model

### Read the CSV files from path


```python
# when the actual data types of given columns cannot be inferred from the first few examples
# they need to be specified manually
# this is where the dtype parameters comes in
df = dd.read_csv(os.path.join(folder_path, '*.csv'), 
                 parse_dates={'Date': [0, 1, 2]},
                 dtype={'TailNum': str,
                        'CRSElapsedTime': float,
                        'Cancelled': bool,
                        'dep_delay': float})
```

## Output

### Calculate the max of a column


```python
# no operation is actually performed until the .compute() function is called
df['dep_delay'].max().compute()
```

### Visualize the parallel execution of the operation


```python
# the underlying task graph can be viewed to understand how the parallel execution takes place
df.dep_delay.max().visualize(rankdir="LR", size="12, 12!")
```

## Comparison

### Pandas


```python
import pandas as pd
import glob
```


```python
%%time
# the equivalent operation performed using Pandas
all_files = glob.glob(os.path.join(folder_path,'*.csv'))
dfs = []
for file in all_files:
    dfs.append(pd.read_csv(file, parse_dates={'Date': [0, 1, 2]}))
df = pd.concat(dfs, axis=0)
df.dep_delay.max()
```

### Dask


```python
%%time
# the entire operation again performed using Dask
df = dd.read_csv(os.path.join(folder_path,'*.csv'), 
                 parse_dates={'Date': [0, 1, 2]},
                 dtype={'TailNum': str,
                        'CRSElapsedTime': float,
                        'Cancelled': bool,
                       'dep_delay': float})
df['dep_delay'].max().compute()

# Dask clearly performs better in comparison to Pandas
# the performance benefits are more apparent when working on larger datasets
# especially when the size of the data exceeds available memory
```


```python

```
