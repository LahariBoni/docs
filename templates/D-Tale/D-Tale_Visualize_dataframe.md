<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/D-Tale/D-Tale_Visualize_dataframe.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=D-Tale+-+Visualize+dataframe:+Error+short+description">🚨 Bug report</a>

**Tags:** #csv #pandas #snippet #read #dataframe #visualize #dtale #operations

**Author:** [Minura Punchihewa](https://www.linkedin.com/in/minurapunchihewa/)

## Input

### Install D-Tale


```python
! pip install --user dtale
```

### Import libraries


```python
import pandas
import dtale
import dtale.app as dtale_app

dtale_app.JUPYTER_SERVER_PROXY = True
```

### Variable


```python
csv_path = "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv"
```

## Model

### Read the CSV from path


```python
df = pandas.read_csv(csv_path)
```

## Output

### Visualize data in file


```python
d = dtale.show(df)
d
```

### Get URL to visualization created by D-Tale


```python
d._main_url
```
