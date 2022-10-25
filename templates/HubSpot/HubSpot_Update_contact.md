<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Update_contact.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Update+contact:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #contact #naas_drivers #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import hubspot
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_KEY = 'YOUR_HUBSPOT_API_KEY'
```

### Enter contact parameters to update


```python
contact_id = "280751"
email = "test@cashstory.com"
firstname = "Jean test"
lastname ='CASHSTOrY'
phone = "+336.00.00.00.00"
jobtitle = "Consultant"
website = "www.cashstory.com"
company = 'CASHSTORY'
hubspot_owner_id = None
```

## Model

### Using patch method


```python
update_contact = {"properties": 
                  {
                    "email": email,
                    "firstname": firstname,
                    "lastname": lastname,
                    "phone": phone,
                    "jobtitle": jobtitle,
                    "website": website,
                    "company": company,
                    "url": "test3",
                    "hubspot_owner_id": hubspot_owner_id,
                   }
                 }

contact_id1 = hubspot.connect(HS_API_KEY).contacts.patch(contact_id,
                                                         update_contact)
```

### Using update method


```python
contact_id2 = hubspot.connect(HS_API_KEY).contacts.update(
    contact_id,
    email,
    firstname,
    lastname,
    phone,
    jobtitle,
    website,
    company,
    hubspot_owner_id
)
```

## Output

### Display results


```python
contact_id1
```


```python
contact_id2
```
