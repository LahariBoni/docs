<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_invitations_received.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+invitations+received:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #content #operations #snippet #invitation #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

In this notebook, you will get all invitations received

## Input

### Import libraries



```python
from naas_drivers import linkedin
import pandas as pd
```

### Setup LinkedIn

- [Get your cookies](/d20a8e7e508e42af8a5b52e33f3dba75)


```python
# Lindekin cookies
LI_AT = "AQEDARCNSioDe6wmAAABfqF-HR4AAAF-xYqhHlYAtSu7EZZEpFer0UZF-GLuz2DNSz4asOOyCRxPGFjenv37irMObYYgxxxxxxx"
JSESSIONID = "ajax:12XXXXXXXXXXXXXXXXX"
```

## Model

### Get invitations
Get the information returned in a dataframe.<br><br>
**Available columns :**
- PROFILE_ID : Sender LinkedIn unique profile id
- PROFILE_URL : Sender LinkedIn profile URL
- PUBLIC_ID : Sender LinkedIn public profile id
- FIRSTNAME : Sender first name
- LASTNAME : Sender last name
- FULLNAME : Sender first and last name
- OCCUPATION : Sender occupation
- PROFILE_PICTURE : Sender profile picture
- MESSAGE : Message sent with the invitation
- SENT_AT : Date time the invitation was sent at (Only for "Profile" invitation)
- UNSEEN : Boolean to know if the invitation have already been seen (Boolean: True or False)
- INVITATION_TYPE : Type of invitation. It could be "Profile", "Company", "Event", "Newsletter"
- INVITATION_DESC : Detailed type of invitation.
- INVITATION_STATUS : Status of the invitation ("PENDING" for invitation from Profile)
- INVITATION_ID : Secret to be used to accept or ignore invitation
- SHARED_SECRET : Secret to be used to accept or ignore invitation


```python
df_invitation = linkedin.connect(LI_AT, JSESSIONID).invitation.get_received()
```

## Output


### Display result


```python
df_invitation
```


```python

```
