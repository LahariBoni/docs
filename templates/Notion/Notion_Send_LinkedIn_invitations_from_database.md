<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Send_LinkedIn_invitations_from_database.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Send+LinkedIn+invitations+from+database:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #invitation #automation #content #linkedin 

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

With this notebook, you will be able to send LinkedIn invitation from a Notion database.<br>
**NB:** Because LinkedIn limit invitations to 100 per week, this notebook only send 10 invites at once.

## Input

### Import libraries


```python
import naas
from naas_drivers import notion, linkedin
import pandas as pd
import os
from datetime import datetime
import requests
```

### Setup Notion
- [Get your Notion integration token](https://docs.naas.ai/drivers/notion)
- Share integration with your database


```python
# Enter Token API
NOTION_TOKEN = "*****"

# Enter Database URL
DATABASE_URL = "https://www.notion.so/********"

# Column with Linkedin URL
col_lk_notion = 'Name'
```

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn cookies
LI_AT = 'YOUR_COOKIE_LI_AT'
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'

# LinkedIn limit invitations to 100 per week, this notebook only send 10 invites at once.
LIMIT = 10
```

### Setup variables


```python
# CSV to manage and remove profile already in your contact
csv_contact = "LINKEDIN_EXISTING_CONTACT.csv"

# CSV to manage URL not valid
csv_not_valid = "LINKEDIN_NOT_VALID.csv"

# CSV to store invitations sent
csv_invitation = "LINKEDIN_INVITATIONS_SENT.csv"
```

### Schedule your notebook


```python
# Scheduler your invitation everyday at 8:00 AM
naas.scheduler.add(cron="0 8 * * *")

# Uncomment the line below to delete your scheduler
# naas.scheduler.delete()
```

## Model

### Get Notion database


```python
db_notion = notion.connect(NOTION_TOKEN).database.get(DATABASE_URL)
df_notion = db_notion.df()
df_notion
```

### Get LinkedIn invitations sent


```python
df_lk_invitations = linkedin.connect(LI_AT, JSESSIONID).invitation.get_sent()
df_lk_invitations
```

### Get profile checked and already in your network


```python
def get_csv(output_path):
    df = pd.DataFrame()
    if os.path.exists(output_path):
        df = pd.read_csv(output_path).drop_duplicates()
    return df
```


```python
df_contacts = get_csv(csv_contact)
df_contacts
```

### Get URL not valid


```python
df_not_valid = get_csv(csv_not_valid)
df_not_valid
```

### Get invitations sent (CSV)
Public ID can be different between what we get from LinkedIn and from your source URL.<br>
So we need to double check invitations sent with a CSV stored on your local


```python
df_csv_invitations = get_csv(csv_invitation)
df_csv_invitations
```

### Get new invitation
- Clean Notion database to get valid URL
- Remove profile when already invited


```python
def get_new_invitations(df_notion,
                        df_lk_invitations,
                        df_csv_invitations,
                        df_contacts,
                        df_not_valid):
    # Cleaning
    df = df_notion.copy()
    df = df[df[col_lk_notion].str.match(".+.com/in/.+")].reset_index(drop=True)
    df["PROFILE_ID"] = df.apply(lambda row: row[col_lk_notion].split("com/in/")[-1].split("/")[0], axis=1)
    print("✔️ LinkedIn valid URL :", len(df))
    
    # Get list of pending LinkedIn invitations
    pending_lk_invitations = []
    if len(df_lk_invitations) > 0:
        pending_lk_invitations = df_lk_invitations["PUBLIC_ID"].unique().tolist()
    print("❌ Pending LinkedIn invitations :", len(pending_lk_invitations))
    
    # Get list of CSV invitations
    pending_csv_invitations = []
    if len(df_csv_invitations) > 0:
        pending_csv_invitations = df_csv_invitations["PUBLIC_ID"].unique().tolist()
    print("❌ Pending CSV invitations :", len(pending_csv_invitations))
    
    # Get profile already in network
    contacts = []
    if len(df_contacts) > 0:
        contacts = df_contacts["PUBLIC_ID"].unique().tolist()
    print("❌ Already in network :", len(contacts))
    
    # Get profile not valid
    not_valids = []
    if len(df_not_valid) > 0:
        not_valids = df_not_valid["PROFILE_ID"].unique().tolist()
    print("❌ Profile not valid:", len(not_valids))
    
    # Remove pending invitations / already in network / not valid profile from dataframe 
    exclude = (pending_lk_invitations + pending_csv_invitations + contacts + not_valids)
    df = df[~df["PROFILE_ID"].isin(exclude)].reset_index(drop=True)
    print("➡️ New invitation:", len(df))
    return df

df_new_invitations = get_new_invitations(df_notion,
                                         df_lk_invitations,
                                         df_csv_invitations,
                                         df_contacts,
                                         df_not_valid)
df_new_invitations
```

### Send invitation


```python
def send_invitation(df,
                    df_not_valid=None,
                    df_contacts=None,
                    df_csv_invitations=None):
    # Check if new invitations to perform
    if len(df) == 0:
        print("🤙 No new invitations to send")
        return df
    
    # Setup variables
    if df_not_valid is None:
        df_not_valid = pd.DataFrame()
    if df_contacts is None:
        df_contacts = pd.DataFrame()
    if df_csv_invitations is None:
        df_csv_invitations = pd.DataFrame()
        
    # Loop
    count = 1
    for index, row in df.iterrows():
        df_network = pd.DataFrame()
        profile = row[col_lk_notion]
        print(f"➡️ Checking :", profile)
        
        # Get distance with profile
        try:
            df_network = linkedin.connect(LI_AT, JSESSIONID).profile.get_network(profile)
        except Exception as e:
            # If error, profile URL is not valid => append Notion page to CSV not valid to not check it again
            df_not_valid = pd.concat([df_not_valid, df[index:index+1]])
            df_not_valid.to_csv(csv_not_valid, index=False)
            print("❌ URL not valid", e)
            
        # Check if profile is already in your network
        if len(df_network) > 0:
            distance = df_network.loc[0, "DISTANCE"]
            # If not profile in network...
            if distance not in ["SELF", "DISTANCE_1"]:
                # => send invitation
                try:
                    linkedin.connect(LI_AT, JSESSIONID).invitation.send(recipient_url=profile)
                    print(count, "- 🙌 Invitation successfully sent")
                    df_csv_invitations = pd.concat([df_csv_invitations, df_network])
                    df_csv_invitations.to_csv(csv_invitation, index=False)
                except Exception as e:
                    print(count, "- ❌ Invitation not sent", e)
                count += 1
            else:
                # If profile already in your network => append network result to CSV existing contact to not check it again
                df_contacts = pd.concat([df_contacts, df_network])
                df_contacts.to_csv(csv_contact, index=False)
                print(f"👍 Already in my network, 💾 saved in CSV")
            
        # Manage LinkedIn limit
        if count > LIMIT:
            print("⚠️ LinkedIn invitation limit reached", LIMIT)
            return df_csv_invitations
    return df_csv_invitations
        
df_csv_invitations = send_invitation(df_new_invitations,
                                     df_not_valid,
                                     df_contacts,
                                     df_csv_invitations)
```

## Output

### Display invitations sent


```python
df_csv_invitations
```


```python

```
