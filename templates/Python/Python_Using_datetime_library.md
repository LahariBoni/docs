<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Using_datetime_library.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Using+datetime+library:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #datetime #snippet #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook will help you use datetime library through several examples.

## Input

### Import library


```python
from datetime import datetime
```

### Setup Variables
- To parse your date format : you can use the [d3 time format documentation](https://github.com/d3/d3-time-format)


```python
# Date string format
date_string = "2022-02-25"

# Your date string format
current_format = "%Y-%m-%d"

# New date format you want to use
new_format = "%Y-W%U"
```

## Model

### Convert a string date to a datetime object


```python
date_datetime = datetime.strptime(date_string, current_format)
date_datetime
```

### Convert a datetime object to string date


```python
new_date = date_datetime.strftime(new_format)
new_date
```

### Convert a string date to a datetime object to another string date


```python
new_date = datetime.strptime(date_string, current_format).strftime(new_format)
new_date
```

## Output

### Display new date


```python
new_date
```
