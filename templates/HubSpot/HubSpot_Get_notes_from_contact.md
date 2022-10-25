<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Get_notes_from_contact.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Get+notes+from+contact:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #sales #crm #engagements #notes #snippet #json #contacts

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This templates will extract all notes from a contact in HubSpot and return a dataframe

## Input

### Import libraries


```python
from datetime import datetime
import requests
import json
import naas
import pandas as pd
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
# Enter Token API
HS_API_KEY = "ENTER_YOUR_HS_API_KEY_HERE" # EXAMPLE : "7865b95b-7731-7843-2537-34284HSKHEZ"
```

### Setup your contact info


```python
# Get contact ID in HubSpot
# If you are in HubSpot, a contact ID is the last part of the URL : https://app.hubspot.com/contacts/XXXX/contact/508201
contact_id = 100001 # EXAMPLE = 100001 or "000001"
```

## Model

### Function to get recent tasks


```python
DATETIME_FORMAT = "%Y-%m-%d %H:%M:%S"

def timestamp_to_date(d):
    result = None
    if d is not None:
        result = datetime.fromtimestamp(int(d) / 1000).strftime(DATETIME_FORMAT)
    return result

def get_notes_from_contact(contact_id):
    url = f'https://api.hubapi.com/engagements/v1/engagements/associated/contact/{contact_id}/paged'
    querystring = {
        "archived": "false",
        "hapikey": HS_API_KEY,
        "limit": 100,
    }
    headers = {'accept': 'application/json'}

    # Get all notes
    df_notes = pd.DataFrame()
    engagements = []
    has_more = True
    offset = None
    while has_more:
        if offset is not None:
            querystring["offset"] = offset
            
        # Requests data
        res = requests.get(url,
                           headers=headers,
                           params=querystring)
        res.raise_for_status()
        res_json = res.json()
        results = res_json.get("results")
        if len(results) > 0:
            for result in results:
                engagement = {}
                associations = {}
                metadata = {}
                engagement = result.get("engagement")
                if len(engagement) > 0:
                    engagement['createdAt'] = timestamp_to_date(engagement.get("createdAt"))
                    engagement['lastUpdated'] = timestamp_to_date(engagement.get("lastUpdated"))
                    engagement['timestamp'] = timestamp_to_date(engagement.get("timestamp"))
                associations = result.get("associations")
                metadata = result.get("metadata")
                engagement.update(associations)
                engagement.update(metadata)
                engagements.append(engagement)
                
        has_more = res_json.get("hasMore")
        offset = res_json.get("offset")
        
    
    df_engagements = pd.DataFrame(engagements)
    if len(df_engagements) > 0:
        df_engagements.columns = df_engagements.columns.str.upper()
        df_notes = df_engagements[df_engagements["TYPE"] == "NOTE"].sort_values("CREATEDAT").reset_index(drop=True)
    print(f"✅ {len(df_notes)} notes fetched from contact {contact_id}")
    return df_notes
```

## Output

### Get all notes


```python
df_notes = get_notes_from_contact(contact_id)
df_notes.head(5)
```
