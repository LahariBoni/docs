<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Analytics/Google_Analytics_Get_unique_visitors_by_country.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Analytics+-+Get+unique+visitors+by+country:+Error+short+description">🚨 Bug report</a>

**Tags:** #googleanalytics #statspercountry #analytics #marketing #dataframe

**Author:** [Charles Demontigny](https://www.linkedin.com/in/charles-demontigny/)

Pre-requisite: Create your own <a href="">Google API JSON credential</a>

## Input

### Import library


```python
import plotly.graph_objects as go
import plotly.express as px
import naas
# Using a dropin driver in a cell for now. (Faster iterations)
# from naas_drivers import googleanalytics
```


```python
%run Google_Analytics_driver.ipynb
```

### Get your credential from Google Cloud Platform


```python
json_path = 'naas-googleanalytics.json'
```

### Get view id from google analytics


```python
view_id = "228952707"
```

## Model

### Get trend for unique visitor by Hourly / Day / Week / Month


```python
df = googleanalytics.connect(json_path=json_path).views.get_data(
    view_id,
    metrics="ga:newUsers",
    pivots_dimensions="ga:country",
    dimensions="ga:date",
    format_type="pivot",
    start_date="30daysAgo",
    end_date="today"
)
df
```

## Output

### Display result


```python
df
```


```python

```
