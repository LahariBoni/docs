<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Data.gouv.fr/COVID19%20-%20%20FR%20-%20Entr%C3%A9es%20et%20sorties%20par%20r%C3%A9gion%20pour%201%20million%20d%27hab..ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Data.gouv.fr+-+COVID19+-++FR+-+Entrées+et+sorties+par+région+pour+1+million+d'hab.:+Error+short+description">🚨 Bug report</a>

**Tags:** #data.gouv.fr #opendata #france #analytics

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

Sur géodes (https://geodes.santepubliquefrance.fr/#c=indicator&view=map2…), aller dans COVID>données hospitalières>Nombre quotidien de nouvelles personnes en réanimation

Récupérer les entrées pour chaque région

Sur géodes, aller dans COVID>données hospitalières>Nombre de personnes actuellement en réanimation.

Récupérer le total pour chaque région

Pour chaque jour et chaque région, calculer :
a) Le solde (Total J - Total J-1)
b) les sorties (entrées - soldes)

Lisser ces données sur les 7 derniers jours

Représenter la courbe des entrées, celles des sorties, le solde entre les 2
Le site de SP : https://santepubliquefrance.fr/maladies-et-traumatismes/maladies-et-infections-respiratoires/infection-a-coronavirus/articles/infection-au-nouveau-coronavirus-sars-cov-2-covid-19-france-et-monde…

Video explicative : https://drive.google.com/file/d/1Mx7pEeH_3puzkDyicvRJ5opgiI3wZJ-d/view <br>
Production : Nairobi Team, 2020/04/24 (MyDigitalSchool)

## Input

### Import libraries


```python
import requests
import pandas as pd
from datetime import datetime, timedelta
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import numpy as np
```

### Variables


```python
# URLs
BASE_URL_ENTREE = 'https://geodes.santepubliquefrance.fr/GC_indic.php?lang=fr&prodhash=3c0e7522&indic=incid_rea&dataset=covid_hospit_incid&view=map2&filters=jour='
BASE_URL_TOTAL = 'https://geodes.santepubliquefrance.fr/GC_indic.php?lang=fr&prodhash=3c0e7522&indic=rea&dataset=covid_hospit&view=map2&filters=sexe=0,jour='

# Liste des départements
DEPARTMENTS = ['Ain', 'Aisne', 'Allier', 'Alpes-de-Haute-Provence', 'Hautes-Alpes', 'Alpes-Maritimes', 'Ardèche', 'Ardennes', 'Ariège', 'Aube', 'Aude', 'Aveyron',
               'Bouches-du-Rhône', 'Calvados', 'Cantal', 'Charente', 'Charente-Maritime', 'Cher', 'Corrèze', 'Côte-d\'Or', 'Côtes-d\'Armor', 'Creuse', 'Dordogne',
               'Doubs', 'Drôme', 'Eure', 'Eure-et-Loir', 'Finistère', 'Corse-du-Sud', 'Haute-Corse', 'Gard','Haute-Garonne','Gers','Gironde','Hérault',
               'Ille-et-Vilaine','Indre','Indre-et-Loire','Isère','Jura','Landes','Loir-et-Cher','Loire','Haute-Loire','Loire-Atlantique','Loiret','Lot',
               'Lot-et-Garonne','Lozère','Maine-et-Loire','Manche','Marne','Haute-Marne','Mayenne','Meurthe-et-Moselle','Meuse','Morbihan','Moselle','Nièvre',
               'Nord','Oise','Orne','Pas-de-Calais','Puy-de-Dôme','Pyrénées-Atlantiques','Hautes-Pyrénées','Pyrénées-Orientales','Bas-Rhin','Haut-Rhin','Rhône',
               'Haute-Saône','Saône-et-Loire','Sarthe','Savoie','Haute-Savoie','Paris','Seine-Maritime','Seine-et-Marne','Yvelines','Deux-Sèvres','Somme','Tarn',
               'Tarn-et-Garonne','Var','Vaucluse','Vendée','Vienne','Haute-Vienne','Vosges','Yonne','Territoire de Belfort','Essonne','Hauts-de-Seine',
               'Seine-Saint-Denis','Val-de-Marne','Val-d\'Oise','Guadeloupe','Martinique','Guyane','La Réunion','Mayotte', 'France Entière']

# Nombre de jours
LISSAGE_JOURS = 7

# Pour contenir les x derniers jours, x étant la variable "LISSAGE_JOURS"
DATES = []

# Les indices contiennent x tableaux ordonnés en fonction de "DATES" contenant les données des départements ordonné comme "DEPARTEMENTS"
INDICES_TEMP_ENTREES = []
INDICES_TEMP_REANIMATION = []
INDICES_TEMP_COURANT = []
INDICES_ENTREES = []
INDICES_COURANT = []
INDICES_SORTIES = []
```

## Model

### Récupération des données


```python
for i in range(LISSAGE_JOURS + 1):
  # Génération des dates
  DAY = (datetime.today() - timedelta(days = (LISSAGE_JOURS - i))).isoformat().split("T")[0]
  DATES.append(DAY)

  # Récupération des entrées en réanimation
  URL = (BASE_URL_ENTREE + DAY)
  RESPONSE = requests.get(URL)
  JSON = RESPONSE.json()
  INDICES_TEMP_ENTREES.append(JSON['content']['distribution']['values'])
  TOTAL_ENTREES = 0
  for value in JSON['content']['distribution']['values']:
    TOTAL_ENTREES += value
    INDICES_ENTREES.append(value)
  INDICES_ENTREES.append(TOTAL_ENTREES)
  
  # Récupération des personnes actuellement en réanimation
  URL = (BASE_URL_TOTAL + DAY)
  RESPONSE = requests.get(URL)
  JSON = RESPONSE.json()
  INDICES_TEMP_REANIMATION.append(JSON['content']['distribution']['values'])
```

### Calcul des données


```python
for i in range(1, LISSAGE_JOURS + 1):
  TOTAL_SORTIES = 0
  for j in range(len(DEPARTMENTS) - 1):
    INDICES_TEMP_COURANT.append([])
    INDICES_TEMP_COURANT[i-1].append(INDICES_TEMP_REANIMATION[i][j] - INDICES_TEMP_REANIMATION[i - 1][j])
    TOTAL_SORTIES += INDICES_TEMP_ENTREES[i][j] - INDICES_TEMP_COURANT[i - 1][j]
    INDICES_SORTIES.append(INDICES_TEMP_ENTREES[i][j] - INDICES_TEMP_COURANT[i - 1][j])
  INDICES_SORTIES.append(TOTAL_SORTIES)

INDICES_ENTREES = INDICES_ENTREES[len(DEPARTMENTS) : len(INDICES_ENTREES)]
DATES.pop(0)

for value in INDICES_TEMP_COURANT:
  TOTAL_COURANT = 0
  for v in value:
    TOTAL_COURANT += v
    INDICES_COURANT.append(v)
  INDICES_COURANT.append(TOTAL_COURANT)
```

### Mise en forme des données


```python
iterables  = []
iterables.append(DATES)
iterables.append(DEPARTMENTS)
idx = pd.MultiIndex.from_product(iterables, names=['DATE', 'ZONE'])

datas = []
for i in range(len(iterables[1]) * len(iterables[0])) :
    datas.append(np.array([INDICES_ENTREES[i], INDICES_SORTIES[i],INDICES_COURANT[i], datetime.today()]))

df = pd.DataFrame(datas, index=idx, columns=['ENTREES', 'SORTIES', 'SOLDES', 'LAST UPDATE'])
df
```

## Output

### Plotting


```python
# Prépare la figure pour deux graphes
fig = make_subplots(rows=2, cols=1)
#fig = go.Figure()

# Application d'un filtre pour le graphe
# df = df.filter(like='France Entière', axis=0)

# Création des éléments du dropdown pour appliquer les filtres
buttons = []
buttons.append(dict(method='restyle', label="Entire France",
                      args=[{'y':[df.filter(like="France Entière", axis=0).ENTREES, df.filter(like="France Entière", axis=0).SORTIES, df.filter(like="France Entière", axis=0).SOLDES]}]))
for i in range(len(DEPARTMENTS) - 1):
  dep = DEPARTMENTS[i]
  buttons.append(dict(method='restyle', label=dep,
                      args=[{'y':[df.filter(like=dep, axis=0).ENTREES, df.filter(like=dep, axis=0).SORTIES, df.filter(like=dep, axis=0).SOLDES]}]))

# Affichage des lignes dans le graphe
fig.add_trace(go.Scatter(x=df.filter(like='France Entière', axis=0).index.get_level_values('DATE'), y=df.filter(like='France Entière', axis=0).ENTREES, fill='tozeroy',name="Admissions",line=dict(width=0.5,color="rgb(160,0,0)"),line_shape='spline'), row = 1, col = 1)
fig.add_trace(go.Scatter(x=df.filter(like='France Entière', axis=0).index.get_level_values('DATE'), y=df.filter(like='France Entière', axis=0).SORTIES, fill='tozeroy',name="Releases",line=dict(width=0.5,color="rgb(0,160,0)"),line_shape='spline'), row = 1, col = 1)
fig.add_trace(go.Scatter(x=df.filter(like='France Entière', axis=0).index.get_level_values('DATE'), y=df.filter(like='France Entière', axis=0).SOLDES, fill='tozeroy',name="Balance",line=dict(width=0.5,color="rgb(0,0,160)"),line_shape='spline'), row = 2, col = 1)

# Redimensionnement et couleur de fond du graphe
fig.update_layout(width=1400, height=400, plot_bgcolor='rgb(255,255,255)', title_text="Admissions, releases and balance for COVID-19 reanimation services in France (last update : " + str(df['LAST UPDATE'][0]) + ")")

# Mise en place du dropdown
fig.update_layout(updatemenus=[dict(buttons=buttons, direction="down", pad={"r": 1, "t": 1}, showactive=True, x=0.05, xanchor="left", y=1.22, yanchor="top")])

fig.update_layout(annotations=[dict(text="Zone", x=0, xref="paper", y=1.18, yref="paper", align="left", showarrow=False)])

# Affichage du graphe
fig.show()
#df
```
