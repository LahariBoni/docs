<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Create_Task.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Create+Task:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #sales #crm #engagements #task #snippet

**Author:** [Alok Chilka](https://www.linkedin.com/in/calok64/)

## Input

### Import libraries


```python
from datetime import datetime, timedelta
import requests
import json
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_TOKEN = "YOUR_HUBSPOT_API_KEY" 
```

### Setup your task info


```python
# Assign owner ID
owner_id = 111111086

#Associated contact ID
asso_contactids=1551

# Time delay to

#Associated contact IDset due date for tasks in days
time_delay = 10


# Task data
subject = "My Third task"
body = "Call contacts"
status = "NOT_STARTED" # NOT_STARTED | COMPLETED | IN_PROGRESS | WAITING | DEFERRED
```

## Model

### Function to create task


```python
def create_task(owner_id,
                time_delay,
                subject,
                body,
                status,
                asso_contactids=[],
                asso_companyids=[],
                asso_dealids=[],
                asso_ownerids=[],
                engagement="TASK"):
    """
    Engagement type = TASK | NOTE | EMAIL | MEETING | CALL 
    """
    
    # Calc timestamp
    tstampobj = datetime.now() + timedelta(days=time_delay)
    tstamp = tstampobj.timestamp() * 1000
     
    # Create payload
    payload = json.dumps({
        "engagement": {
            "active": 'true',
            "ownerId": owner_id,
            "type": engagement,
            "timestamp": tstamp
        },
        "associations": {
            "contactIds": [1551],
            "companyIds": asso_companyids,
            "dealIds": asso_dealids,
            "ownerIds": [owner_id],
        },
        "metadata": {
            "body": body,
            "subject": subject,
            "status": status,
        }
    })
    url = "https://api.hubapi.com/engagements/v1/engagements"
    params = {"hapikey": HS_API_TOKEN}
    headers = {'Content-Type': "application/json"}
    # Post requests
    res = requests.post(url,
                        data=payload,
                        headers=headers,
                        params=params)
    # Check requests
    try:
        res.raise_for_status()
    except requests.HTTPError as e:
        raise (e)
    res_json = res.json()
    
    # Fetch the task id of the current task created
    task_id = res_json.get("engagement").get("id")
    print("🎆 Task created successfully: ", task_id)
    return res_json
```

## Output

### Create task


```python
create_task(owner_id,
            time_delay,
            subject,
            body,
            status)
```


```python

```
