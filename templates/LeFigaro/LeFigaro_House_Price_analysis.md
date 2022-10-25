<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LeFigaro/LeFigaro_House_Price_analysis.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LeFigaro+-+House+Price+analysis:+Error+short+description">🚨 Bug report</a>

**Tags:** #lefigaro #investors #immobilier #markdown #graph #chart #analytics

**Author:** [Mahanamana Andriamiharisoa](https://www.linkedin.com/in/mahanamana/)

As a investor I want to know what is the most interesting investment to be made by number of rooms in a particular area.

## Input

### Import libraries


```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import plotly.express as px
import naas
```

### Setup variables


```python
title = "House price analysis"
lefigaro_url="https://immobilier.lefigaro.fr/annonces/immobilier-vente-maison-paris.html?page=" #link can be changed but keep ?page= to ensure scraping
PAGES = 20 # change the number of pages to scrape

# Output paths
output_image = f"{title}.png"
output_html = f"{title}.html"
```

### Setup scheduler


```python
# if you want to activate or delete the scheduler, uncomment the lines below:

#naas.scheduler.add(cron="0 8 * * *") #every day at 8am
#naas.scheduler.delete() 
```

## Model

### Define functions to get


```python
def get_prices(items):
    prices=[]
    for item in items:
        price = item.select(".price")[0].get_text().strip().replace("€", "").split()
        prices.append("".join(price))
    return prices

def get_location(items):
    locations=[]
    for item in items:
        location=item.select(".title-location")[0].get_text().strip().split(" ")
        location.pop()
        location = "".join(location)
        locations.append(location)
    return locations

def get_vars(items):
    options= []
    piece=[]
    chambre=[]
    surface=[]
    for item in items:
        option = item.select(".options")
        try:
            piece.append(option[0].get_text().strip().split(" ")[0])
        except IndexError:
            piece.append(None)
        try:
            chambre.append(option[1].get_text().strip().split(" ")[0])
        except IndexError:
            chambre.append(None)
        try:
            surface.append(option[2].get_text().strip().split("M")[0])
        except IndexError:
            surface.append(None)
  
    return [piece, chambre, surface]
```

### Use functions to scrape data


```python
for page in range(PAGES):
    url = lefigaro_url+str(page+1)

    whole_html = requests.get(url)

    soup = BeautifulSoup(whole_html.content, "html.parser")
    
    cartouche_list = soup.findAll(True, {"class":"cartouche-liste"})
    
    data = pd.DataFrame({
        "price_(euro)": get_prices(cartouche_list),
        "localisation": get_location(cartouche_list),
        "piece": get_vars(cartouche_list)[0],
        "chambre": get_vars(cartouche_list)[1],
        "surface_(m2)": get_vars(cartouche_list)[2]
    })
    
    if page==0:
        data.to_csv("figaro_dataset.csv", index=False, encoding='utf-8')
        print(str(page+1) + " file creation iteration.")
    elif page != 0:
        data.to_csv("figaro_dataset.csv", mode='a', index=False, encoding='utf-8', header=False)
        print(str(page+1) + " file appending iteration.")
```


```python
df=pd.read_csv("figaro_dataset.csv")
df.dropna(axis=0, inplace=True)
```

### Clean data to create outputs


```python
try:
    df['surface_(m2)']=df['surface_(m2)'].str.split(" ", expand=True)[0]
    df = df[~df['surface_(m2)'].str.contains('[0-9]+[a-zA-Z]+', regex=True)] 
except:
    pass
try:
    df=df[~df['price_(euro)'].str.contains('[^0-9]', regex=True)]
except:
    pass

df['price_(euro)']=pd.to_numeric(df['price_(euro)'], downcast='float', errors='coerce')
df['piece']=pd.to_numeric(df['piece'], errors='coerce')
df['chambre']=pd.to_numeric(df['chambre'], errors='coerce')
df['surface_(m2)']=pd.to_numeric(df['surface_(m2)'], errors='coerce')
df.dropna(axis=0, inplace=True)
```

## Output

### Create chart


```python
labels = {
    "chambre": "Nb. Chambres",
    "piece": "Nb. Pieces",
    "localisation": "Localisation",
    "price_(euro)": "Prix en euro"
}
fig = px.scatter(df[:50],
                 x = "chambre", 
                 y = "price_(euro)",
                 hover_name = 'localisation',
                 color = 'localisation',
                 size = 'price_(euro)',
                 labels = labels,
                 width = 1000, height = 800)

fig.update_layout(title = title)
fig.show()
```

### Export in PNG and HTML


```python
fig.write_image(output_image, width=1200)
fig.write_html(output_html)
```

### Create shareable assets


```python
link_image = naas.asset.add(output_image)
link_html = naas.asset.add(output_html, {"inline": True})
```
