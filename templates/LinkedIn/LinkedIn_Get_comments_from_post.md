<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_comments_from_post.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+comments+from+post:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #post #comments #naas_drivers #content #snippet #dataframe

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

### Enter post URL


```python
POST_URL = "POST_URL"
```

## Model

Get the information return in a dataframe.<br><br>
**Available columns :**
- PROFILE_URN : LinkedIn unique profile id
- PROFILE_ID : LinkedIn public profile id
- FIRSTNAME
- LASTNAME
- TEXT
- OCCUPATION
- ACTIVITY_COMMENTS
- ACTIVITY_LIKES
- DISTANCE
- POST_URL


```python
df = linkedin.connect(LI_AT, JSESSIONID).post.get_comments(POST_URL)
```

## Output

### Display result


```python
df
```
