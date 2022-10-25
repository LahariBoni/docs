<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Send_invitations_to_post_commenters.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Send+invitations+to+post+commenters:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #post #comments #invitations #connections #naas_drivers #content #snippet #dataframe

**Author:** [Asif Syed](https://www.linkedin.com/in/asifsyd/)

## Input

### Import library


```python
from naas_drivers import linkedin
import naas
import time
```

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = naas.secret.get("LI_AT") or 'ENTER_YOUR_COOKIE_LI_AT_HERE'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = naas.secret.get("JSESSIONID") or  'ENTER_YOUR_COOKIE_JSESSIONID_HERE'  # EXAMPLE ajax:8379907400220387585
```

### Enter post URL

To get the URL for a post, click on the three dots option to the right corner of that post on LinkedIn and select "copy link to post" option.

Replace the string 'URL' below with the actual URL


```python
POST_URL = "URL"
```

## Model

### Get comments of the post

Running the below cell fetches the information related to all the comments of the post and stores the information in a dataframe


```python
df = linkedin.connect(LI_AT, JSESSIONID).post.get_comments(POST_URL)
```

### Get profile URLs of all the commenters


```python
profiles = df['PROFILE_URL'] # storing the profile URLs of all the commenters
```

### Filter these profiles to include only people out of network


```python
filtered_profiles = []
for i in profiles:
    distance = linkedin.profile.get_network(profile_url=i)["DISTANCE"]
    if distance[0] != 'DISTANCE_1':
        filtered_profiles.append(i)
    time.sleep(3)
```

## Output

### Send invitations to all the commenters


```python
for i in filtered_profiles: # looping through each commenter's profile URL
    linkedin.connect(LI_AT, JSESSIONID).invitation.send(recipient_url=i) # send invitation
```
