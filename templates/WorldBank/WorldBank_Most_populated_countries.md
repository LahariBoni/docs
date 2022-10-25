<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WorldBank/WorldBank_Most_populated_countries.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WorldBank+-+Most+populated+countries:+Error+short+description">🚨 Bug report</a>

**Tags:** #worldbank #opendata #snippet #plotly #matplotlib

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

**Notebook d'exemple pour classer les pays les plus peuplés**

**Sources:**
OECD -> Organisation for economic co-operation and Development

## Input

### Import library


```python
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
import requests
import io
import numpy as np
import plotly.graph_objects as go
import plotly.express as px
from pydrive.auth import GoogleAuth
from pydrive.drive import GoogleDrive
from google.colab import auth
from oauth2client.client import GoogleCredentials
from pandas import DataFrame
import plotly.graph_objects as go
```

## Model

### Lets search the file frome gdrive


```python
auth.authenticate_user()
gauth = GoogleAuth()
gauth.credentials = GoogleCredentials.get_application_default()
drive = GoogleDrive(gauth)
downloaded = drive.CreateFile({'id':"1FjX4NTIq1z3zS9vCdAdpddtj9mKa0wIW"})   # replace the id with id of file you want to access
downloaded.GetContentFile('POP_PROJ_20042020112713800.csv')
```

### Stock the data in a variable


```python
data = pd.read_csv("POP_PROJ_20042020112713800.csv", usecols=["Country", "Time", "Value"])
data.rename(columns = {'Country':'COUNTRY', 'Time':'TIME', 
                              'Value':'VALUE'}, inplace = True) 
data
```

### Fonction


```python
firstOccur = []
secondOccur = []
firstYear = 2000
secondYear = 2030
def tambouille_first(number1):
  first = []
  for index, row in data.iterrows():
    if(row["TIME"] == number1):
      first.append(row)

  first = DataFrame(first)
  first = first.sort_values(by ="VALUE",ascending=True)
  first = first.tail(10)
  return first

def tambouille_second(number2):
  second = []
  for index, row in data.iterrows():
    if(row["TIME"] == number2):
      second.append(row)

  second = DataFrame(second)
  second =second.sort_values(by ="VALUE",ascending=True)
  second = second.tail(10)
  return second

firstOccur = tambouille_first(firstYear)
secondOccur = tambouille_second(secondYear)

firstOccur

```

## Output

### Display plot


```python
fig = go.Figure(data=[
  go.Bar(name=str(firstYear), y=firstOccur["COUNTRY"], x=firstOccur["VALUE"],orientation='h'),
  go.Bar(name=str(secondYear), y=secondOccur["COUNTRY"], x=secondOccur["VALUE"],orientation='h'),
])
fig.update_layout(title_text="TOP 10 des pays les plus peuplés en 2000 avec prévision 2030", annotations=[
        dict(
            x=1,
            y=-0.15,
            showarrow=False,
            text="Source : OECD -> 2019",
            xref="paper",
            yref="paper"
        )])
fig.show()
```

**Tutorial video (in french)**
https://drive.google.com/file/d/14QhRJTWxlV6HyHmrLuSGsJ6NuFrV2GCZ/view
