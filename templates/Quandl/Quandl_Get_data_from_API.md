<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Quandl/Quandl_Get_data_from_API.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Quandl+-+Get+data+from+API:+Error+short+description">🚨 Bug report</a>

**Tags:** #quandl #marketdata #opendata #finance #snippet #matplotlib

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Install packages


```python
!pip install quandl
```

### Import libraries


```python
import quandl
import matplotlib.pyplot as plt
```

## Model

### Get the data 


```python
data = quandl.get('EIA/PET_RWTC_D')
```

## Output

### Show dataframe


```python
data
```

### Show the graph


```python
%matplotlib inline
data.plot()
```
