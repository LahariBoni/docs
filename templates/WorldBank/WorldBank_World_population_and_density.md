<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WorldBank/WorldBank_World_population_and_density.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WorldBank+-+World+population+and+density:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldbank #opendata #snippet #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

**Objective**

This graph tends to show the population repartition in the world by region. The ordinate measures the growth in population for one year, the abscissa indicates the density, and the cercle shows the number of habitants.

Source
United Nations Population Division.

## Input

### Import library


```python
import pandas as pd
import numpy as np
import plotly.express as px

# Options pour afficher plus de données sur le retour console
# pd.set_option("display.max_rows", 10)
# pd.set_option("display.max_columns", 10)
```

## Model

### Get the data from an excel file


```python
years = list(map(lambda a : str(a), range(1950, 2020, 1)))
usecols = ["Region, subregion, country or area *", "Country code", "Type", *years]
renamed_population_columns = {}
renamed_density_columns = {}

xls_populations = pd.read_excel('https://population.un.org/wpp/Download/Files/1_Indicators%20(Standard)/EXCEL_FILES/1_Population/WPP2019_POP_F01_1_TOTAL_POPULATION_BOTH_SEXES.xlsx',
                    header=16,
                    encoding="utf-8",
                    usecols=usecols)

# Pour chaque année on vient créer une colonne "population_{année}" dans notre dataset
for year in years:
  xls_populations[year] = pd.to_numeric(xls_populations[year], errors='coerce')
  renamed_population_columns[year] = f"population_{year}"
xls_populations = xls_populations.rename(columns=renamed_population_columns)

# On récupère seulement les valeurs du type "Country/Area"
xls_populations = xls_populations[xls_populations['Type'] == "Country/Area"]

xls_populations
```


```python
xls_density = pd.read_excel('https://population.un.org/wpp/Download/Files/1_Indicators%20(Standard)/EXCEL_FILES/1_Population/WPP2019_POP_F06_POPULATION_DENSITY.xlsx',
                    header=16,
                    encoding="utf-8",
                    usecols=["Region, subregion, country or area *", "Country code", "Type", *years])

# Pour chaque année on vient créer une colonne "density_{année}" dans notre dataset
for year in years:
  xls_density[year] = pd.to_numeric(xls_density[year], errors='coerce')
  renamed_density_columns[year] = f"density_{year}"
xls_density = xls_density.rename(columns=renamed_density_columns)

# On récupère seulement les valeurs du type "Country/Area"
xls_density = xls_density[xls_density['Type'] == "Country/Area"]

xls_density

```

### Dataset assembling


```python
# On vient concatener le dataset "Population" avec le dataset "Densité"
result = pd.concat([xls_populations,xls_density], sort=False)
n = result.index.nlevels
xls_global = result.groupby(level=range(n)).first()

xls_global
```

### Adding the dataset


```python
# Pour chaque année on vient comparer la population total d'un pays avec celle de l'année N-1 pour en déduire son évolution sur une année
for index, year in enumerate(years):
  # Suppression des bruits (données non-traitables)
  if index is 0:
    continue
  try:
    past_year = str(int(year) - 1)
    xls_global[f'population_growth_{year}'] = (xls_global[f'population_{year}'] - xls_global[f'population_{past_year}']) / xls_global[f'population_{past_year}'] * 100
  except KeyError:
    xls_global[f'population_growth_{year}'] = np.nan

xls_global
```

### Creating dataset "Continents et leurs pays"



```python
# Récupération des continents via l'API RestCountries
countries = pd.read_json('https://restcountries.eu/rest/v2/all?fields=region;numericCode', dtype = {"numericCode": int})
countries = countries.rename(columns={"region": "Region", "numericCode" : "Country code"})
# Suppression du bruit (données non-traitables)
countries= countries.dropna()
# On format les données pour qu'elles correspondent au format du dataset global
countries['Country code'] = countries['Country code'].replace(regex=r"^0+", value='')
countries["Country code"] = countries["Country code"].astype(int)

countries
```

### Add a column "Région" to the global dataset


```python
xls_global = xls_global.join(countries.set_index('Country code'), on='Country code')

xls_global
```

### Formating the display


```python
# Création de l'ensemble final
xls_formatted = pd.DataFrame(columns=['COUNTRY', 'YEAR', 'POPULATION', 'POPULATION GROWTH', 'DENSITY', 'REGION'])


for index, line in xls_global.iterrows():
  for year in years:
    # On ignore 1950 car il n'est pas possible de calculer l'évolution sans les données de 1949
    if year == "1950":
      continue
    xls_formatted = xls_formatted.append(
        {
            'COUNTRY': line['Region, subregion, country or area *'],
            'YEAR': year,
            'POPULATION': line[f"population_{year}"],
            'POPULATION GROWTH': line[f"population_growth_{year}"],
            'DENSITY': line[f"density_{year}"],
            'REGION': line['Region'],
        }, ignore_index=True)

# Suppression du bruit (données non-traitables)
xls_formatted = xls_formatted.dropna()

xls_formatted
```

## Output

### Display the plot with plotly


```python
fig = px.scatter(xls_formatted, x="DENSITY", y="POPULATION GROWTH", animation_frame="YEAR", animation_group="COUNTRY",
           size="POPULATION", color="REGION", hover_name="COUNTRY",
           log_x=True, size_max=60)
fig.show()
```
