<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_followers_from_network.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+followers+from+network:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #network #followers #naas_drivers #content #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import linkedin
```

### Get your cookies
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585
```

## Model

Note : Due to follower privacy, it will not match with the exact number of followers.<br>

Get the information return in a dataframe.<br><br>
**Available columns :**
- PROFILE_URN : LinkedIn unique profile id
- PROFILE_ID : LinkedIn public profile id
- FIRSTNAME
- LASTNAME
- OCCUPATION
- FOLLOWER_COUNT
- FOLLOWING
- FOLLOWING_TYPE
- INFLUENCER


```python
df = linkedin.connect(LI_AT, JSESSIONID).network.get_followers(limit=-1)
```

## Output

### Display result


```python
df
```
