<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Withdraw_pending_profile_invitations.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Withdraw+pending+profile+invitations:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #invitation #pending #naas_drivers #content #automation

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook removes profile pending invitations.<br>
You can change the month limit by updating the variable in the "Setup Limit" cell below.

## Input

### Import libraries


```python
from naas_drivers import linkedin
import pandas as pd
import naas
import time
from datetime import datetime
from dateutil.relativedelta import relativedelta
import json
import requests
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Credentials
LI_AT = naas.secret.get("LINKEDIN_LI_AT") or "AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2AAXc-FCKmgiMit5FLdY1AAXc-FCKmgiMit5FLdY1"
JSESSIONID = naas.secret.get("LINKEDIN_JSESSIONID") or "ajax:8379907400220XXXXX"
```

### Setup limit


```python
# Invitations pending limit in month(s)
LIMIT = 3
```

### Setup Naas scheduler
For information on changing this setting, please check https://crontab.guru/ for information on the required CRON syntax 


```python
# the default settings below will make the notebook run everyday at 14:00 on Friday
SCHEDULER_CRON = "0 14 * * 5"
```

## Model

### Get invitations sent


```python
df_invitations_sent = linkedin.connect(LI_AT, JSESSIONID).invitation.get_sent()
print("Pending invitations:", len(df_invitations_sent))
df_invitations_sent.head(1)
```

### Filter invitations pending > limit


```python
def out_of_limit(df):
    # Get limit date
    date_limit = datetime.today() - relativedelta(months=LIMIT)
    
    # Create time diff columns
    df.loc[:, "TIME_DIFF"] = 0
    df.loc[pd.to_datetime(df["SENT_AT"]) < date_limit, "TIME_DIFF"] = 1
    
    # Filter on limit exceed
    df = df[df["TIME_DIFF"] == 1]
    return df.reset_index(drop=True)

df_withdraw = out_of_limit(df_invitations_sent)
print("Invitations to withdraw:", len(df_withdraw))
df_withdraw.head(1)
```

### Witdraw invitations sent pending


```python
def withdraw_invitation(invitation_id):
    LinkedIn = linkedin.connect(LI_AT, JSESSIONID)
    cookies = LinkedIn.cookies
    headers = LinkedIn.headers
    payload = {
        "inviteActionType": "ACTOR_WITHDRAW",
        "inviteActionData": [
            {"entityUrn":
             f"urn:li:fs_relInvitation:{invitation_id}",
             "genericInvitation": False,
             "genericInvitationType":"CONNECTION"
            }
        ]
    }
    req_url = "https://www.linkedin.com/voyager/api/relationships/invitations?action=closeInvitations"
    res = requests.post(req_url,
                        data=json.dumps(payload),
                        cookies=cookies,
                        headers=headers)
    res.raise_for_status()
    res_json = res.json()
    return res_json

def withdraw_invitations(df):
    for index, row in df.iterrows():
        fullname = row["FULLNAME"]
        invitation_id = row["INVITATION_ID"]
        print(f"➡️ Withdrawing from invitations pending:", fullname)
        
        # Get distance with profile
        try:
            withdraw_invitation(invitation_id)
        except Exception as e:
            print("❌ Withdraw not available", e)
        time.sleep(3)
    return df

withdraw_invitations(df_withdraw)
```

## Output

### Automate your task


```python
naas.scheduler.add(cron=SCHEDULER_CRON)

# to de-schedule this notebook, simply run the following command: 
# naas.scheduler.delete()
```
