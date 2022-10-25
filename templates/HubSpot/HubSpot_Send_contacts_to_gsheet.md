<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Send_contacts_to_gsheet.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Send+contacts+to+gsheet:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #contact #naas_drivers #gsheet #snippet #googlesheets

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
from naas_drivers import hubspot, gsheet
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_KEY = 'YOUR_HUBSPOT_API_KEY'
```

### Enter contact properties you want to be returned


```python
properties_list = []
```

### Setup your Google Sheet

Pre-requisite: share your Google Sheet with our service account <br>
For the driver to fetch the contents of your google sheet, you need to share it with the service account linked with Naas.<br>
🔗 naas-share@naas-gsheets.iam.gserviceaccount.com


```python
spreadsheet_id = ""
sheet_name = ""
```

## Model

### Get all contacts


```python
df_contacts = hubspot.connect(HS_API_KEY).contacts.get_all(properties_list)
```

## Output

### Send contacts to gsheet


```python
gsheet.connect(spreadsheet_id).send(
    sheet_name=sheet_name,
    data=df_contacts,
    append=False
)
```
