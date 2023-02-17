<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pandas/Pandas_Groupby_and_Aggregate.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pandas+-+Groupby+and+Aggregate:+Error+short+description">Bug report</a>

**Tags:** #pandas #snippet #datamining #dataaggragation #datacleaning #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

**Description:** This notebook groups and perform aggregation on columns.

<u>References:</u> 
- https://towardsdatascience.com/5-pandas-group-by-tricks-you-should-know-in-python-f53246c92c94
- https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.groupby.html
- https://www.w3resource.com/pandas/dataframe/dataframe-agg.php
- https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.agg.html

## Input

### Import libraries


```python
import pandas as pd
```

### Setup DataFrame


```python
# create DataFrame
df = pd.DataFrame(
    {
        "team": ["A", "A", "A", "B", "B", "B"],
        "points": [11, 7, 8, 10, 21, 13],
        "assists": [5, 7, 7, 9, 12, 0],
        "rebounds": [5, 8, 10, 6, 6, 22],
    }
)
df
```

### Define columns to group


```python
# list of columns to group
to_group = ["team"]
```

### Define columns to aggregate
Accepted combinations are:
- function
- string function name: "sum", "count", "mean"
- list of functions and/or function names, e.g. [np.sum, 'mean']
- dict of axis labels -> functions, function names or list of such.

Here, we are going to use dict of axis labels -> functions


```python
# dict of columns to aggregate
to_agg = {
    "points": "sum",
    "assists": "sum",
    "rebounds": "sum",
}
```

## Model

### Groupby and Aggregate columns
- When you use .groupby() function on any categorical column of DataFrame, it returns a GroupBy object. Then you can use different methods on this object and even aggregate other columns to get the summary view of the dataset.
- The agg() method allows you to apply a function or a list of function names to be executed along one of the axis of the DataFrame, default 0, which is the index (row) axis.


```python
# For aggregated output, return object with group labels as the index.
# Only relevant for DataFrame input. as_index=False is effectively “SQL-style” grouped output.

df = df.groupby(by=to_group, as_index=False).agg(to_agg)
```

## Output

### View new DataFrame


```python
df
```