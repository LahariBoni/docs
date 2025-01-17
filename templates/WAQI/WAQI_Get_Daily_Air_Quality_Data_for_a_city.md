<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WAQI/WAQI_Get_Daily_Air_Quality_Data_for_a_city.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WAQI+-+Get+Daily+Air+Quality+Data+for+a+city:+Error+short+description">Bug report</a>

**Tags:** #waqi #airquality #api #data #city #python

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/jeremyravenel/)

**Description:** This notebook will demonstrate how to use the WAQI API to get daily air quality data for a city.

**References:**
- [WAQI API Documentation](https://aqicn.org/api/)
- [WAQI API Token](https://aqicn.org/data-platform/token/)

## Input

### Import libraries


```python
import requests
import json
```

### Setup Variables
- `token`: WAQI API token. [Get your token here](https://aqicn.org/data-platform/token/).
- `city`: City name.


```python
token = "YOUR_TOKEN_HERE"
city = "Paris"
```

## Model

### Get daily air quality data

This function will use the WAQI API to get daily air quality data for a city.


```python
def get_daily_air_quality_data(token, city):
    url = f"https://api.waqi.info/feed/{city}/?token={token}"
    response = requests.get(url)
    data = json.loads(response.text)
    return data
```

## Output

### Display result


```python
data = get_daily_air_quality_data(token, city)
print(data)
```

 
