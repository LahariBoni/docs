<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Get_contact_from_URL.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=bug_report.md&title=">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #contact #naas_drivers #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

Get contact from HubSpot URL

## Input

### Import library


```python
from naas_drivers import hubspot
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
# Enter Token API
HS_API_KEY = "ENTER_YOUR_HS_API_KEY_HERE" # EXAMPLE : "7865b95b-7731-7843-2537-34284HSKHEZ"
```

### Enter your contact id


```python
contact_url =  "ENTER_CONTACT_URL_HERE" # EXAMPLE : "https://app.hubspot.com/contacts/0000011/contact/00001"
contact_id = contact_url.split("/contact/")[-1]
```

## Model

### Get single contact


```python
contact = hubspot.connect(HS_API_KEY).contacts.get(contact_id)
```

## Output

### Display result


```python
contact
```
