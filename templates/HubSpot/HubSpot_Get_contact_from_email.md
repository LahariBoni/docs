<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Get_contact_from_email.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Get+contact+from+email:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #contact #naas_drivers #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This template get contact from HubSpot with email.

## Input

### Import libraries


```python
import requests
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
# Enter Token API
HS_API_KEY = "ENTER_YOUR_HS_API_KEY_HERE" # EXAMPLE : "7865b95b-7731-7843-2537-34284HSKHEZ"
```

### Enter your contact email


```python
contact_email = "ENTER_CONTACT_EMAIL_HERE" # EXAMPLE : "florent@naas.ai"
```

## Model

### Get single contact


```python
def get_contact_hubspot(hs_value, hs_key="email", hs_properties=None):
    url = f"https://api.hubapi.com/crm/v3/objects/contacts/{hs_value}"
    querystring = {
        "archived": "false",
        "hapikey": HS_API_KEY,
        "idProperty": hs_key}
    if hs_properties is not None:
        querystring["properties"] = hs_properties
    headers = {'accept': 'application/json'}
    # Requests data
    res = requests.get(url,
                       headers=headers,
                       params=querystring)
    res.raise_for_status()
    return res.json()

contact = get_contact_hubspot(contact_email)
```

## Output

### Display result


```python
contact
```
