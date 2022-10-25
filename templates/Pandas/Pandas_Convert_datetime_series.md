<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pandas/Pandas_Convert_datetime_series.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pandas+-+Convert+datetime+series:+Error+short+description">🚨 Bug report</a>

**Tags:** #pandas #python #date #conversion #operations #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook will help you understand to convert a datetime column to any other datetime format.

## Input

### Import library


```python
import pandas as pd
```

### Create Sample Dataframe 


```python
dict1 = {
    "Name": ["Peter","Dolly","Maggie","David","Isabelle"],
    "Date": ["20/2/2021","19/8/2014","8/9/2000","4/3/2013","14/7/1995"],
    "Second Date": ["August 20,2011","September 16,1993","January 23,2009","October 17,2019","March 4,2021"]
}
df = pd.DataFrame(dict1)
df
```

### Setup Variables
- To parse your date format : you can use the [d3 time format documentation](https://github.com/d3/d3-time-format)


```python
# Your date string format
current_format = "%d/%m/%Y"

# New date format you want to use
new_format = "%Y-W%U"
```

## Model

### Convert datetime string series to datetime series to another datetime string


```python
df["New_Date"] = pd.to_datetime(df["Date"], format=current_format).dt.strftime(new_format)
df
```

## Output

### Display new dataframe


```python
df
```
