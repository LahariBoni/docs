<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_info_from_company.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+info+from+company:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #company #info #naas_drivers #content #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import librairy


```python
from naas_drivers import linkedin
```

### Get your cookies
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585
```

### Enter company URL, linkedin name or id


```python
COMPANY_URL = "https://www.linkedin.com/company/naas-ai" #or universal name = "tesla" or id = "8819091"
```

## Model

Get the information return in a dataframe.<br><br>
**Available columns :**
- COMPANY_ID
- COMPANY_URL
- COMPANY_NAME
- UNIVERSAL_NAME
- LOGO_URL
- INDUSTRY_URN
- WEBSITE
- TAGLINE
- SPECIALITIES
- DESCRIPTION
- COUNTRY
- REGION
- CITY
- STAFF_COUNT
- STAFF_RANGE
- FOLLOWER_COUNT


```python
df = linkedin.connect(LI_AT, JSESSIONID).company.get_info(COMPANY_URL)
```

## Output

### Display result


```python
df
```
