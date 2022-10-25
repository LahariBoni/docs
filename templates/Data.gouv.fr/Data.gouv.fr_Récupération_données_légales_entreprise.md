<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Data.gouv.fr/Data.gouv.fr_R%C3%A9cup%C3%A9ration_donn%C3%A9es_l%C3%A9gales_entreprise.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Data.gouv.fr+-+Récupération+données+légales+entreprise:+Error+short+description">🚨 Bug report</a>

**Tags:** #data.gouv.fr #snippet #naas #societe #opendata

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
import requests
from pprint import pprint
```

### Setup Variables


```python
# Input
SIREN = "877941724"
```

## Model

### Récupération des données


```python
def get_info(siren):
    req_url = f"https://entreprise.data.gouv.fr/api/sirene/v3/unites_legales/{siren}"
    res = requests.get(req_url)
    res.raise_for_status
    return res.json()
    
data = get_info(SIREN)
```

## Output

### Display result


```python
pprint(data)
```
