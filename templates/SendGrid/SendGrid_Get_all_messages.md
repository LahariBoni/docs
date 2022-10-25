<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/SendGrid/SendGrid_Get_all_messages.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=SendGrid+-+Get+all+messages:+Error+short+description">🚨 Bug report</a>

**Tags:** #sendgrid #activity #snippet #operations #dataframe

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)

This notebook enables you to get a dataframe of all the activities performed

## Input

### Imports


```python
import naas
import requests
import urllib
import pandas as pd
```

### Setup SendGrid
👉 Get your [api key](https://app.sendgrid.com/settings/api_keys)


```python
SENDGRID_API_KEY = naas.secret.get("SENDGRID_API")
```

## Model

### Get activity
https://docs.sendgrid.com/api-reference/e-mail-activity/filter-all-messages?code-sample=code-filter-all-messages&code-language=Python&code-sdk-version=6.x


```python
def get_messages(msg_id=None,
                 from_email=None,
                 subject=None,
                 to_email=None,
                 status=None,
                 clicks=None,
                 limit=1000):
    
    kargs = locals()
    params = {}
    for k in kargs:
        v = kargs.get(k)
        if v is not None:
            params[k] = v
    req_url = f"https://api.sendgrid.com/v3/messages?{urllib.parse.urlencode(params)}"
    headers = {
        "Authorization": f"Bearer {SENDGRID_API_KEY}",
        "Content-Type": "application/json"
    }
    res = requests.get(req_url,
                       headers=headers)
    try:
        res.raise_for_status()
    except requests.HTTPError as e:
        raise(e)
    res_json = res.json()
    messages = res_json.get("messages")
    
    # Formatting
    df = pd.DataFrame(messages)
    df["last_event_time"] = df["last_event_time"].astype(str).str.replace("T", " ").str.replace("Z", "")
    df.columns = df.columns.str.upper()
    return df

df_messages = get_messages(limit=1000)
```

## Output

### Display result


```python
df_messages
```
