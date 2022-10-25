<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_connections_from_network.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+connections+from+network:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #network #connections #naas_drivers #analytics #csv #html #image #content #plotly

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import linkedin
import pandas as pd
from datetime import datetime
import naas
import plotly.graph_objects as go
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585
```

### Variables


```python
# Outputs
title = "LinkedIn - Connections"
name_output = "LinkedIn_connections"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

## Model

### Get connections from LinkedIn network
**Available columns :**
- FIRSTNAME : First name
- LASTNAME : Last name
- OCCUPATION : Text below the name in the profile page
- CREATED_AT : Date of connection between and your connection
- PROFILE_URL : Profile URL
- PROFILE_PICTURE : Profile picture URL
- PROFILE_ID : LinkedIn profile id
- PUBLIC_ID : LinkedIn public profile id


```python
df_connections = linkedin.connect(LI_AT, JSESSIONID).network.get_connections(limit=-1)
df_connections
```

## Output

### Save connections to CSV


```python
df_connections.to_csv(csv_output, index=False)
```

### Share outputs with naas


```python
naas.asset.add(csv_output)

#-> to remove your outputs, uncomment the lines and execute the cell
# naas.asset.delete(csv_output)
```
