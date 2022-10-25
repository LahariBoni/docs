<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Accept_all_invitations_and_send_first_message.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Accept+all+invitations+and+send+first+message:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #content #operations #automation #invitation

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

In this notebook, you will be able to automically accept all invitations from LinkedIn

## Input

### Import libraries



```python
from naas_drivers import linkedin
import naas
import pandas as pd
```

### Setup LinkedIn

- [Get your cookies](/d20a8e7e508e42af8a5b52e33f3dba75)


```python
# Lindekin cookies
LI_AT = "AQEDARCNSioDe6wmAAABfqF-HR4AAAF-xYqhHlYAtSu7EZZEpFer0UZF-GLuz2DNSz4asOOyCRxPGFjenv37irMObYYgxxxxxxx"
JSESSIONID = "ajax:12XXXXXXXXXXXXXXXXX"

# First message
FIRST_MESSAGE = "Hello, Nice to connect!"
```

### Setup Naas


```python
# Schedule your notebook every hour
naas.scheduler.add(cron="0 * * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get invitations received


```python
df_invitation = linkedin.connect(LI_AT, JSESSIONID).invitation.get_received()
df_invitation
```

### Accept pending invitations received from "Profile"


```python
def accept_new_contact(df):
    df_accept = pd.DataFrame()
    
    # Loop
    for index, row in df.iterrows():
        fullname = row.FULLNAME
        status = row.INVITATION_STATUS
        invitation_id = row.INVITATION_ID
        shared_secret = row.SHARED_SECRET
        if status == "PENDING":
            print(fullname)
            tmp_df = linkedin.connect(LI_AT, JSESSIONID).invitation.accept(invitation_id, shared_secret)
            df_accept = pd.concat([df_accept, tmp_df])
    return df_accept

df_accept = accept_new_contact(df_invitation)
df_accept
```

### Send first message to contact


```python
def send_first_message(df):
    # Loop
    for index, row in df.iterrows():
        fullname = row.FULLNAME
        profile_id = row.PROFILE_ID
        print(fullname)
        linkedin.connect(LI_AT, JSESSIONID).message.send(FIRST_MESSAGE, profile_id)

send_first_message(df_accept)
```

## Output


### Display result



```python
df_accept
```
