<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Get_Task.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Get+Task:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #sales #crm #engagements #task #snippet #json

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
contact_id = 1551
owner_id = 111111086

# Time delay to get tasks created since N days, where N is no of days. For ex. Get tasks created since 1 day
time_delay = 10

#Number of tasks to be retrieved
no_of_tasks = 10
```

## Model

### Function to get recent tasks


```python
def get_task(contact_id,owner_id,time_delay,no_of_tasks):
    """
    Engagement type = TASK  
    """
    
    # Calc timestamp
    Previous_Date = datetime.now() - timedelta(days=time_delay)
    Previous_tstamp = Previous_Date.timestamp() * 1000
    Previous_tstamp = math.trunc(Previous_tstamp)
     
    
    url = "https://api.hubapi.com/engagements/v1/engagements/recent/modified"
    params = {"hapikey": HS_API_TOKEN,"since":Previous_tstamp,"count":no_of_tasks}
    headers = {'Content-Type': "application/json"}
    # Post requests
    res = requests.get(url,headers=headers,params=params)
    
    if res.status_code == 200:
        
        res_json = res.json()

        # Check requests
        try:
            res.raise_for_status()
        except requests.HTTPError as e:
            raise (e)
        res_json = res.json()

        return res_json
    else:
        print("Task not found")
   
```

## Output

### Get Recent task


```python
results = get_task(contact_id,owner_id,time_delay,no_of_tasks)
```


```python
for key in results["results"]:
    print("---------------")
    print(key['engagement']['id'])
```


```python

```
