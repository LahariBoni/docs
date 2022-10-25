<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/OpenWeatherMap/OpenWeatherMap_Send_daily_email_with_predictions.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=OpenWeatherMap+-+Send+daily+email+with+predictions:+Error+short+description">🚨 Bug report</a>

**Tags:** #openweathermap #weather #plotly #prediction #email #naas_drivers #automation #opendata #analytics #ai #image #html #text

**Author:** [Gautier Vivard](https://www.linkedin.com/in/gautier-vivard-1811b877/)

## Input

### Import libraries


```python
import requests
import markdown2
import time
import pandas as pd
import naas
from naas_drivers import plotly, prediction
```

### Setup your open weather info


```python
OPENWEATHER_KEY = '***************'  # get your key from here https://home.openweathermap.org/api_keys (it takes couples of minutes)
city = 'rouen'
country_code = 'fr' # if you don't want to specify a country code, let ''
```


```python
# Output paths image and html
output_image = f'{city}.png'
output_html = f'{city}.html'
```

### Input email parameter


```python
email_to = ["template@naas.ai"]
email_from = None
subject = f'{city} predictions as of today'
```

### Schedule every day


```python
# naas.scheduler.add(cron='0 8 * * *')
# naas.scheduler.delete()
```

### Create markdown template


```python
%%writefile message.md
Hey

The *CITY* temperature on the last 5 days

In +2 days, basic ML models predict the following temperature: 

- *linear*: LINEAR

    
<img href=link_html target="_blank" src=link_image style="width:640px; height:360px;" /><br>
[Open dynamic chart](link_html)<br>

    
Have a nice day.
<br>

PS: You can [send the email again](link_webhook) if you need a fresh update.<br>
<div><strong>Full Name</strong></div>
<div>Open source lover | <a href="http://www.naas.ai/" target="_blank">Naas</a></div>
```

### Add email template as a dependency 


```python
naas.dependency.add("message.md")
```

## Model

### Get the data from open weather map

The historical open weather api need the latitude, longitude in order to have the data


```python
def get_geoloc(city: str, country_code: str = ''):
    """ Get the geoloc of a city, country
    
    :param city: name of the city
    :type city: str
    :param country_code: Please use ISO 3166 country codes, default to ''
    :type country_code: str
    """
    url = f'http://api.openweathermap.org/geo/1.0/direct?q={city},,{country_code}&appid={OPENWEATHER_KEY}'
    return requests.get(url).json()

def get_lat_lon(city: str, country_code: str = ''):
    """ Get the geoloc of a city, country
    
    :param city: name of the city
    :type city: str
    :param country_code: Please use ISO 3166 country codes, default to ''
    :type country_code: str
    """
    geoloc = get_geoloc(city, country_code)
    
    if len(geoloc) == 0:
        return None, None
    
    return geoloc[0]['lat'], geoloc[0]['lon']

# get_lat_lon('paris')
# get_lat_lon('paris', 'us')
```


```python
def get_historical_weather(city: str, country_code: str = '', nbr_days_before_now: int = 0):
    """Get historical weather data. For free API, maximum history is 5 days before now
    
    :param city: name of the city
    :type city: str
    :param country_code: Please use ISO 3166 country codes, default to ''
    :type country_code: str 
    :param nbr_hours_before_now: number of hour before now
    """
    unix_dt = int(time.time() - 60 * 60 * 24 * nbr_days_before_now)
    lat, lon = get_lat_lon(city, country_code)
    if lat is None:
        return None
    url = f'https://api.openweathermap.org/data/2.5/onecall/timemachine?lat={lat}&lon={lon}&dt={unix_dt}&appid={OPENWEATHER_KEY}&units=metric'
    return requests.get(url).json()

def weather_data_to_df(city: str, country_code: str = '', nbr_days_before_now: int = 0) -> pd.DataFrame:
    data = get_historical_weather(city, country_code, nbr_days_before_now) 
    df = pd.DataFrame(data['hourly'])
    df['date_time'] = pd.to_datetime(df['dt'], unit='s')
    df['city'] = city
    df['country_code'] = country_code
    
    df_explode_weather = pd.concat([df.drop(['weather', 'dt'], axis=1), df['weather'].str[0].apply(pd.Series)], axis=1)
    # df_explode_weather.set_index('date_time', inplace=True)
    return df_explode_weather
```


```python
df_histo_weather = pd.concat([weather_data_to_df(city, country_code, _) for _ in range(6)], ignore_index=True)
df_histo_weather = df_histo_weather.sort_values(by='date_time').reset_index(drop=True).rename(columns={"date_time": "Date"})
df_histo_weather
```

### Add prediction column


```python
df_predict = prediction.get(dataset=df_histo_weather,
                            date_column='Date',
                            column="temp",
                            data_points=5,
                            prediction_type="all")

df_predict
```

### Build chart


```python
chart = plotly.linechart(df_predict,
                         x='Date',
                         y=['temp', 'ARIMA', "LINEAR", "SVR", "COMPOUND"],
                         showlegend=True,
                         title=f'Temp in {city} last 5 days')
```

## Output

### Save as html and png


```python
chart.write_html(output_html)
chart.write_image(output_image, width=1200)
```

### Expose chart


```python
link_image = naas.asset.add(output_image)
link_html = naas.asset.add(output_html, {'inline': True})
```

### Add webhook to run your notebook again


```python
link_webhook = naas.webhook.add()
```

### Create email content


```python
markdown_file = "message.md"
content = open(markdown_file, "r").read()
md = markdown2.markdown(content)
md
```


```python
post = md.replace("DATANOW", str(DATANOW))
post = post.replace("CITY", str(city))
post = post.replace("LINEAR", str(LINEAR))
post = post.replace("link_image", str(link_image))
post = post.replace("link_html", str(link_html))
post = post.replace("link_webhook", str(link_webhook))
post
```

### Send email


```python
content = post
naas.notification.send(email_to=email_to,
                       subject=subject,
                       html=content,
                       files=files,
                       email_from=email_from)
```
