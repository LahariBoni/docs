<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Delete_note.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Delete+note:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #sales #crm #engagements #note #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This template deletes note with a note ID as input

## Input

### Import libraries


```python
import requests
import json
import naas
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
# Enter Token API
HS_API_KEY = "ENTER_YOUR_HS_API_KEY_HERE" # EXAMPLE : "7865b95b-7731-7843-2537-34284HSKHEZ"
```

### Setup your note info


```python
note_id = 19996680052
```

## Model

### Function to delete note


```python
def delete_note(note_id):
    url = f'https://api.hubapi.com/crm/v3/objects/notes/{note_id}'
    headers = {
            "accept": "application/json",
            "content-type": "application/json",
        }
    params = {"limit": "100",
              "archived": "false",
              "hapikey": HS_API_KEY}
    # Requests data
    res = requests.delete(url,
                          headers=headers,
                          params=params)
    res.raise_for_status()
    if res.status_code == 204:
        print(f"❎ Note deleted in HubSpot: {note_id}")
```

## Output

### Delete note


```python
delete_note(note_id)
```
