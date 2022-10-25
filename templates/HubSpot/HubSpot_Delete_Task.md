<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Delete_Task.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Delete+Task:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #sales #crm #engagements #task #snippet

**Author:** [Alok Chilka](https://www.linkedin.com/in/calok64/)

## Input

### Import libraries


```python
from datetime import datetime, timedelta
import requests, math
import json
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_TOKEN = "YOUR_HUBSPOT_API_KEY" 
```

### Setup your task info


```python
task_id = 19996680052
```

## Model

### Function to get recent tasks


```python
def delete_task(uid):
    #set headers
    params = {"hapikey": HS_API_TOKEN}
    headers = {'Content-Type': "application/json"}
    
    #check if tasks already exist
    get_url = "https://api.hubapi.com/engagements/v1/engagements/"
    get_res = requests.get(
                url=f"{get_url}/{uid}",
                headers= headers,
                params= params,
                allow_redirects=False,    
    )
    if get_res.status_code == 200:
        print("Task found..Deleting")
        #delete task
        del_url = "http://api.hubapi.com/engagements/v1/engagements/"
    
        # Post requests
        res = requests.delete(
                url=f"{url}/{uid}",
                headers= headers,
                params= params,
                allow_redirects=False,
            )
    else:
        print("Task not found. Unable to delete")
```

## Output

### Delete task


```python
delete_task(task_id)
```


```python

```
