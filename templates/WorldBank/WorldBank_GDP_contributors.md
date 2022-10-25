<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WorldBank/WorldBank_GDP_contributors.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WorldBank+-+GDP+contributors:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldbank #opendata #snippet #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import libraries


```python
import pandas as pd
from pandas_datareader import wb
import plotly.graph_objects as go
```

## Model

### Data recovery


```python
country = wb.get_countries()["iso2c"]
startYear = 2017
endYear = startYear + 1

#Recupération des valeurs pour World (Initial - Final) GDP/PPL
data_gdp_world_2017 = wb.download(indicator=['NY.GDP.PCAP.KD.ZG'], country=['WLD'], start=startYear, end=startYear)
data_gdp_world_2018 = wb.download(indicator=['NY.GDP.PCAP.KD.ZG'], country=['WLD'], start=endYear, end=endYear)

gdp_growth_2017 = data_gdp_world_2017.iloc[0][0]
gdp_growth_2018 = data_gdp_world_2018.iloc[0][0]

data_gdp_current_year = wb.download(indicator=['NY.GDP.PCAP.KD.ZG'], country=country, start=endYear, end=endYear)

data_gdp_current_year
```

### Sort the data


```python
data_gdp_biggest = data_gdp_current_year.sort_values('NY.GDP.PCAP.KD.ZG', ascending=False)
data_gdp_biggest = data_gdp_biggest.head(5)

data_gdp_lowest = data_gdp_current_year.sort_values('NY.GDP.PCAP.KD.ZG', ascending=True)
data_gdp_lowest = data_gdp_lowest.head(5)
```

### Formattage des données


```python
#Data formating

measure = []
text = []
x = []
y = []


#Push initial
measure.append("absolute")
text.append(str(round(gdp_growth_2017,4)))
y.append(gdp_growth_2017)
x.append(str(startYear) + " Growth")


#Push Beetween
# measure.append("relative")
# text.append(str(round(gdp_growth_2018 - gdp_growth_2017,4)))
# y.append(gdp_growth_2018 - gdp_growth_2017)
# x.append("Between")

totalBiggest = 0

for index, row in data_gdp_biggest.iterrows() :
    measure.append("relative")
    text.append(index[0])
    totalBiggest += row[0]
    y.append(row[0])
    x.append(index[0])


totalLowest = 0

for index, row in data_gdp_lowest.iterrows() :
    measure.append("relative")
    text.append(index[0])
    totalLowest += row[0]
    y.append(row[0])
    x.append(index[0])


#Other Country total
otherCountryTotal =   gdp_growth_2018 - (gdp_growth_2017 + totalLowest + totalBiggest)
measure.append("relative")
text.append(str(round(otherCountryTotal,4)))
y.append(otherCountryTotal)
x.append("Other Country")



#Push result
measure.append("total")
text.append(str(round(gdp_growth_2018,4)))
y.append(gdp_growth_2018)
x.append(str(endYear) + " Growth")


```

## Output

### Plot display

Quand on passe le curseur sur chaque partie du graphique, on peut voir le PIB de l'année courrante, l'année précédente et la différence entre les deux du pays sélectionné.

Si la différence est positive, une flèche montante apparaite sinon une flèche descendante.

Au dessus de chaque pays, le pourcentage de croissance est représenté.


```python
fig = go.Figure(go.Waterfall(
    name = "Growth between Year", orientation = "v",
    measure = measure,
    x = x,
    text = text,
    y = y,
    textposition = "outside",
    texttemplate="%{y:.2f}%",
    connector = {"line":{"color":"rgb(63, 63, 63)"}},
))

fig.update_layout(title = "GDP growth with 5 top and lowest contributors (per capita per country) ", showlegend = True, margin=dict(l=0, r=0, t=50, b=0),
    height=700)

fig.show()
```

### Refaire les étapes pour le PIB par pays


```python
#Recupération des valeurs pour World (Initial - Final) GDP/country
data_gdp_world_2017 = wb.download(indicator=['NY.GDP.MKTP.KD.ZG'], country=['WLD'], start=startYear, end=startYear)
data_gdp_world_2018 = wb.download(indicator=['NY.GDP.MKTP.KD.ZG'], country=['WLD'], start=endYear, end=endYear)

gdp_growth_pll_2017 = data_gdp_world_2017.iloc[0][0]
gdp_growth_2018 = data_gdp_world_2018.iloc[0][0]

data_gdp_current_year = wb.download(indicator=['NY.GDP.MKTP.KD.ZG'], country=country, start=endYear, end=endYear)


data_gdp_current_year
```


```python
data_gdp_biggest = data_gdp_current_year.sort_values('NY.GDP.MKTP.KD.ZG', ascending=False)
data_gdp_biggest = data_gdp_biggest.head(5)

data_gdp_lowest = data_gdp_current_year.sort_values('NY.GDP.MKTP.KD.ZG', ascending=True)
data_gdp_lowest = data_gdp_lowest.head(5)

```


```python
#Data formating

measure = []
text = []
x = []
y = []


#Push initial
measure.append("absolute")
text.append(str(round(gdp_growth_2017,4)))
y.append(gdp_growth_2017)
x.append(str(startYear) + " Growth")


#Push Beetween
# measure.append("relative")
# text.append(str(round(gdp_growth_2018 - gdp_growth_2017,4)))
# y.append(gdp_growth_2018 - gdp_growth_2017)
# x.append("Between")

totalBiggest = 0

for index, row in data_gdp_biggest.iterrows() :
    measure.append("relative")
    text.append(index[0])
    totalBiggest += row[0]
    y.append(row[0])
    x.append(index[0])


totalLowest = 0

for index, row in data_gdp_lowest.iterrows() :
    measure.append("relative")
    text.append(index[0])
    totalLowest += row[0]
    y.append(row[0])
    x.append(index[0])


#Other Country total
otherCountryTotal =   gdp_growth_2018 - (gdp_growth_2017 + totalLowest + totalBiggest)
measure.append("relative")
text.append(str(round(otherCountryTotal,4)))
y.append(otherCountryTotal)
x.append("Other Country")



#Push result
measure.append("total")
text.append(str(round(gdp_growth_2018,4)))
y.append(gdp_growth_2018)
x.append(str(endYear) + " Growth")
```


```python
fig = go.Figure(go.Waterfall(
    name = "Growth between Year", orientation = "v",
    measure = measure,
    x = x,
    text = text,
    y = y,
    textposition = "outside",
    texttemplate="%{y:.2f}%",
    connector = {"line":{"color":"rgb(63, 63, 63)"}},
))

fig.update_layout(title = "GDP growth with 5 top and lowest contributors (per country)", showlegend = True,margin=dict(l=0, r=0, t=50, b=0), height=700)

fig.show()
```
