<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_messages_from_profile.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+messages+from+profile:+Error+short+description">Bug report</a>

**Tags:** #linkedin #message #naas_drivers #content #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

**Description:** This notebook allows you to access and respond to messages sent to your LinkedIn profile.

## Input

### Import library


```python
from naas_drivers import linkedin
```

### Get your cookies
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = "YOUR_COOKIE_LI_AT"  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = "YOUR_COOKIE_JSESSIONID"  # EXAMPLE ajax:8379907400220387585
```

### Enter conversation URL


```python
CONVERSATION_URL = "CONVERSATION_URL"
```

## Model


```python
df = linkedin.connect(LI_AT, JSESSIONID).message.get_messages(CONVERSATION_URL)
```

Get the information return in a dataframe.<br><br>
**Available columns :**
- MESSAGE_URN
- SENDER_URN
- CREATED_AT
- MESSAGE_BODY
- PROFILE_URN
- FIRSTNAME
- LASTNAME
- OCCUPATION

## Output

### Display result


```python
df
```
