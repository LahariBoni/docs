<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Send_event_invitations_post_engagements.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Send+event+invitations+post+engagements:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #events #invitations #naas_drivers #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook send invitations to your event from your post engagements.

*NB: You must to organized the event to be able to send invitations!*<br>
*NB2: You will be able to send invitations only to people you are connected!*

## Input

### Import library


```python
from naas_drivers import linkedin
import pandas as pd
import naas
import requests
import time
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Credentials
LI_AT = naas.secret.get("LI_AT") or 'ENTER_YOUR_COOKIE_LI_AT_HERE'  # EXAMPLE: AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = naas.secret.get("JSESSIONID") or 'ENTER_YOUR_COOKIE_JSESSIONID_HERE'  # EXAMPLE: ajax:8379907400220387585

# Event URL
EVENT_URL = "ENTER_YOUR_EVENT_URL_HERE" # EXAMPLE: https://www.linkedin.com/events/XXXXXXXXXXXX/

# Post URL
POST_URL = "ENTER_YOUR_POST_URL_HERE" # EXAMPLE: https://www.linkedin.com/posts/XXXXXXXXXXXX/
```

## Model

### Get list of events attendees


```python
df_attendees = linkedin.connect(LI_AT, JSESSIONID).event.get_guests(EVENT_URL)
print("✅ Attendees fetched:", len(df_attendees))
df_attendees.head(1)
```

### Get post likes and comments


```python
def get_engagements(post_url):
    # Get engagements
    df_likes = linkedin.connect(LI_AT, JSESSIONID).post.get_likes(post_url)
    df_comments = linkedin.connect(LI_AT, JSESSIONID).post.get_comments(post_url)
    
    # Concat
    df = pd.concat([df_likes, df_comments]).drop_duplicates("PROFILE_ID", keep="last").reset_index(drop=True)
    return df

df_engagements = get_engagements(POST_URL)
print("✅ Engagements fetched:", len(df_engagements))
df_engagements.head(1)
```

### Get new invitations


```python
def get_new_invitations(df_attendees, df_engagements):
    attendees_list = df_attendees.PROFILE_ID.unique()
    df = df_engagements[~df_engagements.PROFILE_ID.isin(attendees_list)]
    return df

df_new_invitations = get_new_invitations(df_attendees, df_engagements)
print("✅ New invitations fetched:", len(df_new_invitations))
df_new_invitations.head(1)
```

## Output

### Send invitations to event


```python
LinkedIn = linkedin.connect(LI_AT, JSESSIONID)
cookies = LinkedIn.cookies
headers = LinkedIn.headers

def send_invitations(profile_id, event_url):
    event_id = event_url.split("/events/")[-1].split("/")[0]
    payload = {"invitations":
               [{"emberEntityName":"growth/invitation/norm-invitation",
                 "invitee":{"com.linkedin.voyager.growth.invitation.GenericInvitee":
                            {"inviteeUrn": f"urn:li:fs_miniProfile:{profile_id}"}},
                 "trackingId":"4T5tesDXQDqC9ArO15TLag==",
                 "inviterUrn": f"urn:li:fs_professionalEvent:{event_id}"}],
               "defaultCountryCode":""}
    req_url = f"https://www.linkedin.com/voyager/api/growth/normInvitations?action=batchCreate"
    res = requests.post(req_url,
                        cookies=cookies,
                        headers=headers,
                        json=payload)
    res.raise_for_status()
    return res
```

### Send invitation


```python
def send_invitation(df):
    # Check if new invitations to perform
    if len(df) == 0:
        print("🤙 No new invitations to send")
        return df
    
    # Loop
    for index, row in df.iterrows():
        profile = row["FULLNAME"]
        profile_id = row["PROFILE_ID"]
        print(f"➡️ Checking :", profile, profile_id)
        
        # Get distance with profile
        try:
            send_invitations(profile_id, EVENT_URL)
            print(index, "- 🙌 Invitation successfully sent")
        except Exception as e:
            print("❌ Invitation not sent", e)
        time.sleep(3)
            
send_invitation(df_new_invitations)
```
