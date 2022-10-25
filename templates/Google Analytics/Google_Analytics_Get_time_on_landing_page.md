<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Analytics/Google_Analytics_Get_time_on_landing_page.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Analytics+-+Get+time+on+landing+page:+Error+short+description">🚨 Bug report</a>

**Tags:** #googleanalytics #timeonlanding #marketing #analytics #automation #image #csv #html #plotly

**Author:** [Charles Demontigny](https://www.linkedin.com/in/charles-demontigny/)

Pre-requisite: Create your own <a href="">Google API JSON credential</a>

## Input

### Import library


```python
from datetime import timedelta
import pandas as pd
import plotly.graph_objects as go
import naas
from naas_drivers import googleanalytics
```

### Get your credential from Google Cloud Platform


```python
json_path = 'naas-googleanalytics.json'
```

### Get view id from google analytics


```python
view_id = "228952707"
```

### Schedule your notebook


```python
naas.scheduler.add(cron="0 8 * * *")
naas.dependency.add(json_path)

#-> Uncomment the line below (by removing the hashtag) to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Time spent on landing page


```python
df_avg_time = googleanalytics.connect(json_path=json_path).views.get_time_landing(view_id=view_id, landing_path="/")
df_avg_time
```

## Output

### Display result


```python
avg_time_on_landing['avg_time_landing']
```

### Plot Time spent on landing page


```python
def gen_tickvals(avg_time_landing: pd.Series) -> tuple:
    """
    Generate tick text and values.
    """
    delta = int(avg_time_landing.std() / 3)
    minimum = int(avg_time_landing.min() - delta)
    maximum = int(avg_time_landing.max() + delta)
    tickvals = list(range(minimum, maximum, delta))
    ticktext = [str(timedelta(seconds=v)) for v in tickvals]
    return tickvals, ticktext

def plot_time_spent_on_landing(df: pd.DataFrame):
    """
    Plot time spent on landing page in Plotly.
    """
    tickvals, ticktext = gen_tickvals(df['avg_time_landing'])
    data = go.Scatter(
        x=pd.to_datetime(df['Year Month'] + "01"),
        y=df['avg_time_landing']
    )

    fig = go.Figure(data=data)
    fig.update_traces(mode='lines+markers')
    fig.update_layout(title="Avg. Time on Landing Page (in seconds)", template="none")
    fig.update_yaxes(ticktext=ticktext, tickvals=tickvals)
    return fig
```


```python
plot_time_spent_on_landing(avg_time_on_landing)
```


```python
avg_time_on_landing['avg_time_landing']
```


```python
str(timedelta(seconds=70))
```


```python

```
