<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_profile_information.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+profile+information:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #profile #resume #naas_drivers #content #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import linkedin
import naas
```

### Get your cookies
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = naas.secret.get("LINKEDIN_LI_AT")
JSESSIONID = naas.secret.get("LINKEDIN_JSESSIONID")
```

### Enter profile URL


```python
PROFILE_URL = "florent-ravenel"
```

## Model

### Get identity


```python
df_identity = linkedin.connect(LI_AT, JSESSIONID).profile.get_identity(PROFILE_URL)
df_identity
```

### Get network


```python
df_network = linkedin.connect(LI_AT, JSESSIONID).profile.get_network(PROFILE_URL)
df_network
```

### Get contact


```python
df_contact = linkedin.connect(LI_AT, JSESSIONID).profile.get_contact(PROFILE_URL)
df_contact
```

### Get resume


```python
df_resume = linkedin.connect(LI_AT, JSESSIONID).profile.get_resume(PROFILE_URL)
df_resume
```

## Output

### Display result
