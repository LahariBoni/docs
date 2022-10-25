<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Analytics/Google_Analytics_Get_stats_per_country.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Analytics+-+Get+stats+per+country:+Error+short+description">🚨 Bug report</a>

**Tags:** #googleanalytics #statspercountry #marketing #analytics #automation #image #csv #html #plotly

**Author:** [Charles Demontigny](https://www.linkedin.com/in/charles-demontigny/)

Pre-requisite: Create your own <a href="">Google API JSON credential</a>

## Input

### Import library


```python
try:
    import pycountry
except:
    !pip install pycountry
    import pycountry
import plotly.graph_objects as go
import plotly.express as px
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

### Visitor's country of origin


```python
df_country = googleanalytics.connect(json_path=json_path).views.get_data(
    view_id,
    metrics="ga:sessions",
    pivots_dimensions="ga:country",
    dimensions="ga:month",
    start_date=None,
    end_date=None,
    format_type="pivot"
)
df_country
```


```python
sessions_per_country = googleanalytics.connect(json_path=json_path).views.get_country(view_id) # default: metrics="ga:sessions"
```


```python
sessions_per_country
```


```python
users_per_country = googleanalytics.views.get_country(view_id, metrics="ga:users")  
```

## Output

### Display result


```python
sessions_per_country.head()
```


```python
users_per_country.head()
```


```python
sessions_per_country = sessions_per_country.reset_index().rename(columns={"index": "Country"})
mapping = {country.name: country.alpha_3 for country in pycountry.countries}
sessions_per_country['iso_alpha'] = sessions_per_country['Country'].apply(lambda x: mapping.get(x))
```


```python
sessions_per_country
```


```python
fig = px.choropleth(sessions_per_country, locations="iso_alpha",
                    color="Sessions", 
                    hover_name="Country",
                    color_continuous_scale="Greens")
fig.show()
```


```python

```
