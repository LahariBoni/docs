<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Send_invitation_to_company_followers.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Send+invitation+to+company+followers:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #company #followers #invitations #naas_drivers #automation #content

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import linkedin
import pandas as pd
from datetime import datetime
import naas
import plotly.graph_objects as go
import os
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Credentials
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585

# Company URL
COMPANY_URL = "https://www.linkedin.com/company/naas-ai/"

# LinkedIn limit invitations up to 100 per week (Becareful !)
LIMIT = 10
```

### Setup variables


```python
# Inputs
csv_input = "LinkedIn_company_followers.csv"

# Outputs
csv_contact = "LINKEDIN_EXISTING_CONTACT.csv" # CSV to manage and remove profile already in your contact
csv_not_valid = "LINKEDIN_NOT_VALID.csv" # CSV to manage URL not valid
csv_invitation = "LINKEDIN_INVITATIONS_SENT.csv" # CSV to store invitations sent
```

### Setup Naas


```python
# Schedule your notebook everyday at 9:00 AM
naas.scheduler.add(cron="0 9 * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get followers from company


```python
# Get company followers from CSV stored in your local (Returns empty if CSV does not exist)
def get_company_followers(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_followers = get_company_followers(csv_input)
df_followers
```


```python
def get_new_followers(df, input_path):
    # Get all profiles
    profiles = []
    if len(df) > 0:
        profiles = df.PROFILE_ID.unique()
    start = 0
    while True:
        tmp_df = linkedin.connect(LI_AT, JSESSIONID).company.get_followers(COMPANY_URL,
                                                                           start=start,
                                                                           limit=1,
                                                                           sleep=False)
        profile_id = None
        if "PROFILE_ID" in tmp_df.columns:
            profile_id = tmp_df.loc[0, "PROFILE_ID"]
        if profile_id in profiles:
            break
        else:
            df = pd.concat([tmp_df, df])
            df.to_csv(input_path, index=False)
            start += 1
    return df.reset_index(drop=True)

merged_df = get_new_followers(df_followers, csv_input)
merged_df
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
def get_new_invitations(df,
                        df_lk_invitations,
                        df_csv_invitations,
                        df_contacts):
    
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
    
    # Remove pending invitations / already in network / not valid profile from dataframe 
    exclude = (pending_lk_invitations + pending_csv_invitations + contacts)
    df = df[~df["PUBLIC_ID"].isin(exclude)].dropna().reset_index(drop=True)
    print("➡️ New invitation:", len(df))
    return df

df_new_invitations = get_new_invitations(merged_df,
                                         df_lk_invitations,
                                         df_csv_invitations,
                                         df_contacts)
df_new_invitations
```

### Send invitation


```python
def send_invitation(df,
                    df_contacts=None,
                    df_csv_invitations=None):
    # Check if new invitations to perform
    if len(df) == 0:
        print("🤙 No new invitations to send")
        return df
    
    # Setup variables
    if df_contacts is None:
        df_contacts = pd.DataFrame()
    if df_csv_invitations is None:
        df_csv_invitations = pd.DataFrame()
        
    # Loop
    count = 1
    df.PROFILE_ID = df.PROFILE_ID.fillna(0)
    for index, row in df.iterrows():
        df_network = pd.DataFrame()
        profile = row["PUBLIC_ID"]
        print(f"➡️ Checking :", profile)
        
        # Get distance with profile
        if profile != 0:
            df_network = linkedin.connect(LI_AT, JSESSIONID).profile.get_network(profile)
            
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
                    print("❌ Invitation not sent", e)
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
                                     df_contacts,
                                     df_csv_invitations)
```

## Output

### Save company followers in CSV


```python
if len(merged_df) > 0:
    merged_df.to_csv(csv_input, index=False)
    naas.dependency.add(csv_input)
```

### Display invitations sent


```python
df_csv_invitations
```


```python

```
