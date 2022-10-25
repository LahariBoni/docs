<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Locate_Addresses.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Locate+Addresses:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #snippet #naas #geocoder

**Author:** [Suhas B](https://www.linkedin.com/in/suhasbrao/)

This notebook allows you to easily find the ***latitude and longitude coordinates for any address or to find an address from any set of coordinates***

## Input

### Import libraries


```python
try:
    import geocoder
except:
    !pip install geocoder --user
    import geocoder
```

### Setup Variables


```python
# sample address to geo code
address = "Sayyaji Rao Rd, Agrahara, Chamrajpura, Mysuru, Karnataka 570001"
```

## Model

### Get data from geocoder


```python
geo = geocoder.arcgis(address)
```

## Output

### Display latitude and longitude from address


```python
latitude_and_longitude = geo.latlng
print(f"Latitude and Longitude of given address: {latitude_and_longitude}")

# If we want to retrieve the location from a set of coordinates
# perform a reverse query.

location_from_coordinates = geocoder.arcgis(latitude_and_longitude, method="reverse")
print(f"Location from coordinates: {location_from_coordinates[0]}")
```
