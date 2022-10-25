<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Update_linkedinbio_from_google.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Update+linkedinbio+from+google:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #contact #naas_drivers #googlesearch #scheduler #naas #automation

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import hubspot
import naas
import pandas as pd
from googlesearch import search
import time
import re
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_KEY = 'YOUR_HUBSPOT_API_KEY'
```

### Setup Naas


```python
naas.scheduler.add(cron="0 8 * * *")

# -> Uncomment the line below (by removing the hashtag) and execute this cell to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get all contacts in Hubspot


```python
properties_list = [
    "hs_object_id",
    "firstname",
    "lastname",
    "linkedinbio",
]
hubspot_contacts = hubspot.connect(HS_API_KEY).contacts.get_all(properties_list)
hubspot_contacts
```

### Filter to get linkedinbio "Not Defined" and "firstname" and "lastname" defined


```python
df_to_update = hubspot_contacts.copy()

# Cleaning
df_to_update = df_to_update.fillna("Not Defined")

# Filter on "Not defined"
df_to_update = df_to_update[(df_to_update.firstname != "Not Defined") & 
                            (df_to_update.lastname != "Not Defined") &
                            (df_to_update.linkedinbio == "Not Defined")].reset_index(drop=True)

df_to_update
```

### Search bio in Google with firstname and lastname


```python
def get_bio(firstname, lastname):
    # Init linkedinbio
    linkedinbio = None
    
    # Create query
    query = f"{firstname}+{lastname}+Linkedin"
    print("Google query: ", query)
    
    # Search in Google
    for i in search(query, tld="com", num=10, stop=10, pause=2):
        pattern = "https:\/\/.+.linkedin.com\/in\/.([^?])+"
        result = re.search(pattern, i)

        # Return value if result is not None
        if result != None:
            linkedinbio = result.group(0).replace(" ", "")
            return linkedinbio
        else:
            time.sleep(2)
    return linkedinbio
```


```python
for _, row in df_to_update.iterrows():
    firstname = row.firstname
    lastname = row.lastname
    
    # Get linkedinbio
    linkedinbio = get_bio(firstname, lastname)
    df_to_update.loc[_, "linkedinbio"] = linkedinbio
    
df_to_update
```

## Output

### Update linkedinbio in Hubspot


```python
for _, row in df_to_update.iterrows():
    # Init data
    data = {}
    
    # Get data
    hs_object_id = row.hs_object_id
    linkedinbio = row.linkedinbio

    # Update LK Bio
    if linkedinbio != None:
        data = {"properties": {"linkedinbio": linkedinbio}}
    hubspot.connect(HS_API_KEY).contacts.patch(hs_object_id, data)
```
