<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Create_contact.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Create+contact:+Error+short+description">🚨 Bug report</a>

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

### Enter contact parameters


```python
email = "test@cashstory.com"
firstname = "Test"
lastname ='CASHSTORY'
phone = "+33600000000"
jobtitle = "Consultant"
website = "www.cashstory.com"
company = 'CASHSTORY'
hubspot_owner_id = None
```

## Model

### Create contact

#### Using send method


```python
create_contact = {"properties": 
                  {
                    "email": email,
                    "firstname": firstname,
                    "lastname": lastname,
                    "phone": phone,
                    "jobtitle": jobtitle,
                    "website": website,
                    "company": company,
                    "url": "test",
                    "hubspot_owner_id": hubspot_owner_id,
                   }
                 }

contact1 = hubspot.connect(HS_API_KEY).contacts.send(create_contact)
```

### Using create method


```python
contact2 = hubspot.connect(HS_API_KEY).contacts.create(
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
contact1
```


```python
contact2
```
