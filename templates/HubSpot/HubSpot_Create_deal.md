<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Create_deal.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Create+deal:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #deal #naas_drivers #snippet

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

### Enter deal parameters


```python
dealname = "TEST"
closedate = None #must be in format %Y-%m-%d
amount = None
hubspot_owner_id = None
```

### Enter deal stage ID


```python
df_pipelines = hubspot.connect(HS_API_KEY).pipelines.get_all()
df_pipelines
```


```python
dealstage = '5102584'
```

## Model

### Create deal

### Using send method


```python
send_deal = {"properties": 
                  {
                    "dealstage": dealstage,
                    "dealname": dealname,
                    "amount": amount,
                    "closedate": closedate,
                    "hubspot_owner_id": hubspot_owner_id,
                   }
                 }

deal1 = hubspot.connect(HS_API_KEY).deals.send(send_deal)
```

### Using create method


```python
deal2 = hubspot.connect(HS_API_KEY).deals.create(
    dealname,
    dealstage,
    closedate
)
```

## Output

### Display results


```python
deal1
```


```python
deal2
```
