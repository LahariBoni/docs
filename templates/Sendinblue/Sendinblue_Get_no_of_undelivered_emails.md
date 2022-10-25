<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Sendinblue/Sendinblue_Get_no_of_undelivered_emails.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Sendinblue+-+Get+no+of+undelivered+emails:+Error+short+description">🚨 Bug report</a>

**Tags:** #emails #campaign #undelivered #emailcampaigns #marketing #sendinblue #operations #snippet

**Author:** [Minura Punchihewa](https://www.linkedin.com/in/minurapunchihewa/)

This notebook get the number of undelivered emails

## Input

### Import libraries


```python
import requests
import json
import naas
```

### Setup Sendinblue
[How to get your Sendinblue API KEY?](https://developers.sendinblue.com/docs#:~:text=Generate%20your%20API%20key%20to,key%20%60api%2Dkey%60.)


```python
SENDINBLUE_API_KEY = naas.secret.get(name="SENDINBLUE_API_KEY") or "ENTER_YOUR_SENDINBLUE_API_KEY"
```

### Variable


```python
campaign_name = "Minura's Campaign"
```

## Model

### Send request to Sendinblue API


```python
response = requests.get(
    'https://api.sendinblue.com/v3/emailCampaigns',
    headers={
        'Accept': 'application/json',
        'api-key': SENDINBLUE_API_KEY
    }
)
```

## Output

### Get the campaign


```python
campaign = next(item for item in json.loads(response.text)['campaigns'] if item["name"] == campaign_name)
```

### Get the number of undelivered emails: soft bounce + hard bounce


```python
campaign['statistics']['globalStats']['softBounces'] + campaign['statistics']['globalStats']['hardBounces']
```
