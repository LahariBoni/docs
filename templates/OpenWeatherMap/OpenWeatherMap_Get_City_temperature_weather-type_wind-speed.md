<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/OpenWeatherMap/OpenWeatherMap_Get_City_temperature_weather-type_wind-speed.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=OpenWeatherMap+-+Get+City+temperature+weather-type+wind-speed:+Error+short+description">🚨 Bug report</a>

**Tags:** #openweathermap #opendata #snippet #dataframe

**Author:** [Kanishk Pareek](https://in.linkedin.com/in/kanishkpareek)

This notebook helps to get the temperature and wind speed of your city by only giving the city as input.

## Input
### Import library


```python
import requests
```

### Setup Variables


```python
OPENWEATHER_KEY = "**********" # get your key from here https://home.openweathermap.org/api_keys (it takes couples of minutes)
CITY = "Paris"
```

## Model


```python
def get_temperature(json_data):
    temp_in_celcius = json_data['main']['temp']
    return temp_in_celcius

def get_weather_type(json_data):
    weather_type = json_data['weather'][0]['description']
    return weather_type

def get_wind_speed(json_data):
    wind_speed = json_data['wind']['speed']
    return wind_speed

def get_weather_data(json_data, city):
    description_of_weather = json_data['weather'][0]['description']
    weather_type = get_weather_type(json_data)
    temperature = get_temperature(json_data)
    wind_speed = get_wind_speed(json_data)
    weather_details = ''
    return weather_details + ("The weather in {} is currently {} with a temperature of {} degrees and wind speeds reaching {} km/ph".format(city, weather_type, temperature, wind_speed))

def run(city):
    url = f'https://api.openweathermap.org/data/2.5/weather?q={city}&appid={OPENWEATHER_KEY}&units=metric'
    json_data = requests.get(url).json()
    weather_details = get_weather_data(json_data, city)
    print(weather_details)
```

## Output

### Display result


```python
run(CITY)
```
