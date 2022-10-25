<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Generate_leads_from_posts.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Generate+leads+from+posts:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #post #comments #naas_drivers #content #snippet #dataframe

**Author:** [Alok Chilka](https://www.linkedin.com/in/calok64/)

## Input

### Import libraries


```python
from naas_drivers import linkedin, hubspot
import pandas as pd
import numpy as np
import naas
from datetime import datetime, timedelta
import requests
import json
```

### Setup your LinkedIn
- Get your cookies : <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Lindekin cookies
LI_AT = "AQEFALsBAAAAAAbjCtIAAAF_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
JSESSIONID = "ajax:42778xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Enter your profile URL
PROFILE_URL = "<YOUR_LINKEDIN_PROFILE_URL>"
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)<br>
👉 Get your HubSpot owner ID<br>
👉 Get your HubSpot company ID from URL


```python
# HubSpot API key
HS_API_TOKEN = "<YOUR_HUBSPOT_API_TOKEN>"

# Your company located after "reports-dashboard/" when you are connected in HubSpot https://app.hubspot.com/reports-dashboard/2474088/view/244930
HS_COMPANY_ID = "<YOUR_HUBSPOT_COMPANY_ID>"

#Hubspot contact URL
HS_CONTACT_URL = "https://app.hubspot.com/contacts/"+HS_COMPANY_ID+"/contact/"

# Contact owner => contact id to whom task needs to be assigned
HS_OWNER_ID = "<YOUR_HUBSPOT_API_TOKEN>" #remove double quotes from owner id (to be added as integer value)

# Time delay to set due date for tasks in days
time_delay = 10
    
# Calc timestamp
tstampobj = datetime.now() + timedelta(days=time_delay)
tstamp = tstampobj.timestamp() * 1000
     
hs = hubspot.connect(HS_API_TOKEN)
```

### Setup your email to receive notifications


```python
SEND_EMAIL_TO = "<YOUR_EMAIL_ID>"
```

## Model

### Get posts from LinkedIn feed


```python
df_posts = linkedin.connect(LI_AT, JSESSIONID).profile.get_posts_feed(PROFILE_URL, count=100)
df_posts.head()
```

### Get likes from LinkedIn posts


```python
def get_likes(df_posts):
    DF_all_post_likes = pd.DataFrame()
    for index, row in df_posts.iterrows():
        df = linkedin.connect(LI_AT, JSESSIONID).post.get_likes(row['POST_URL'])
        DF_all_post_likes = DF_all_post_likes.append(df)
```


```python
get_likes(df_posts)
```

### Get number of likes by profile


```python
def count_likes(df):
    to_group = ["PROFILE_URN",
                "PROFILE_ID",
                "FIRSTNAME",
                "LASTNAME"]
    #df = df.groupby(to_group, as_index=False).agg({"PROFILE_ID": "count"}).reset_index(name='count')
    #df = df.sort_values(by="PROFILE_ID", ascending=False)
    df2 = df.groupby(['PROFILE_ID']).size().sort_values(ascending=False).reset_index(name='LIKE_COUNT')
    return df2

df_counts = count_likes(DF_all_post_likes)
df_counts
```

### Apply classification to get potential lead

- Grouping the like count and classify the leads as per levels i.e. <b>L1, L2, L3, L4.</b> ( from highest to lowest counts of likes) with column name <b>"POTENTIAL LEAD"</b>
- Group posts as per the the column <b>"POTENTIAL LEAD"</b>


```python
max_likes = df_counts["LIKE_COUNT"].max()
cluster_1 = round(max_likes * 0.1, 0)
cluster_2 = round(max_likes * 0.5, 0)
cluster_3 = round(max_likes * 0.8, 0)
```


```python
conditions = [
    (df_counts["LIKE_COUNT"] <= cluster_1),
    (df_counts["LIKE_COUNT"] > cluster_1) & (df_counts["LIKE_COUNT"] <= cluster_2),
    (df_counts["LIKE_COUNT"] > cluster_2) & (df_counts["LIKE_COUNT"] <= cluster_3),
    (df_counts["LIKE_COUNT"] > cluster_3)
    ]

values = ['L4', 'L3', 'L2', 'L1']
```


```python
df_counts['POTENTIAL_LEAD'] = np.select(conditions, values)
df_counts
```

### Get profile details based on the POST_URL above
- filter rows from "grouped_profile_posts" dataframe based on L1, L2, & L3 levels and add data to dataframe "df_leads"
- Iterate through dataframe "df_leads" and add items to leads_list
- Extract EMAIL, FIRSTNAME, LASTNAME, PHONE_NUMBER, OCCUPATION from profile URL


```python
df_leads = df_counts.loc[df_counts['POTENTIAL_LEAD'].isin(["L1", "L2"])]
df_leads
```


```python
leads_list = [];

r_count = 1

for index, row in df_leads.iterrows():
    profileid = row['PROFILE_ID']
    profileurl = "https://www.linkedin.com/in/"+profileid+"/"
       
    PROFILECONTACTS = linkedin.connect(LI_AT, JSESSIONID).profile.get_contact(profileurl)
    PROFILEIDENTITY = linkedin.connect(LI_AT, JSESSIONID).profile.get_identity(profileurl)
        
    profileemail = PROFILECONTACTS.at[0,'EMAIL']
    profilephoneno = PROFILECONTACTS.at[0,'PHONENUMBER']
    profilename = PROFILEIDENTITY.at[0,'FIRSTNAME'] + " "+ PROFILEIDENTITY.at[0,'LASTNAME']
    profilefirstname = PROFILEIDENTITY.at[0,'FIRSTNAME']
    profilelastname = PROFILEIDENTITY.at[0,'LASTNAME']
    profileoccupation = PROFILEIDENTITY.at[0,'OCCUPATION']
    leads_list.append([profilename, profilefirstname, profilelastname, profileemail, profilephoneno, profileoccupation, profileurl])
    r_count = r_count + 1
```

### Create hubspot contacts from linkedin likes

<h4>HS_CONTACT_ID_LIST</h4> - The list to store the contact id recently created and the linkedin profile URL of associated contact


```python
#to store the resulting output of create contact method
contact_id = ""
HS_CONTACT_ID_LIST = []
for i in leads_list:
        profilename = i[0]
        profilefirstname = i[1]
        profilelastname = i[2]
        profileemail = i[3]
        profilephoneno = i[4]
        profileoccupation = i[5]
        profileurl = i[6]
        # With send method
        data = {"properties": 
                {
                    "linkedinbio": profileurl,
                    "firstname": profilefirstname,
                    "lastname": profilelastname,
                    "jobtitle": profileoccupation,
                    "email": profileemail,
                    "phone": profilephoneno,
                     "hubspot_owner_id": 111111086,
                }
               }
        #write data to CRM ( create new contact in CRM)
       
        contact_id = hs.contacts.send(data)
        HS_CONTACT_ID_LIST.append([contact_id, profileurl])
```

<h4>Build Email Template</h4> 
- This includes the hubspot contact_id, contact URL and LinkedIN URL


```python
table_header = f'<table style="border-collapse:collapse;border-spacing:0;font-family:Arial"><thead><tr><th style="border-color:black;border-style:solid;border-width:1px;padding:6px 10px;text-align:left">Contact ID</th><th style="border-color:black;border-style:solid;border-width:1px;padding:6px 10px;text-align:left">Hubspot URL</th><th style="border-color:black;border-style:solid;border-width:1px;padding:6px 10px;text-align:left">LinkedIN URL</th></tr></thead><tbody>'
table_body = ''
table_footer = '</tbody></table>'

for i in HS_CONTACT_ID_LIST:
    hs_contact_url = HS_CONTACT_URL + i[0]
    linkedin_url = i[1]
    tablerow = '<tr><td style="border-color:black;border-style:solid;border-width:1px;padding:6px 10px;">'+i[0]+'</td><td style="border-color:black;border-style:solid;border-width:1px;padding:6px 10px;">'+hs_contact_url+'</td><td style="border-color:black;border-style:solid;border-width:1px;padding:6px 10px;">'+linkedin_url+'</td></tr>'
    table_body = table_body + tablerow

table = table_header + table_body + table_footer

email_body = f'<p style="font-family:Arial">Hi there,</p><br/><p style="font-family:Arial">Following new task(s) has been created for you :</p><br/>'+table
```

## Output

### Create tasks for user and send email notification


```python
def create_task(owner_id,
                tstamp,
                contact_id,contact_props,contact_linkedin_url,hs_contact_url,engagement="TASK"):
    """ owner_id=HS_OWNER_ID, tstamp=tstamp, subject=subject, body=body, status=status, engagement="TASK"
    Engagement type = TASK | NOTE | EMAIL | MEETING | CALL 
    """
   
    payload = json.dumps({
            "engagement": {
                "active": 'true',
                "ownerId": 111111086,
                "type": "TASK",
                "timestamp": tstamp
            },
            "associations": {
                "contactIds": [1551],
                "companyIds": [],
                "dealIds": [],
                "ownerIds": [],
            },

            "metadata": {
                "body": "Hi there, you need to contact following user & task is already assigned to you.<br/>" + "Name :" +contact_props['firstname']+ contact_props['lastname'] + " Contact URL : " + hs_contact_url,
                "subject": "Task created for Contact ID :"+ contact_id,
                "status": "NOT_STARTED",
                "forObjectType": "CONTACT"
            }
        });
    url = "https://api.hubapi.com/engagements/v1/engagements"
    params = {"hapikey": HS_API_TOKEN}
    headers = {'Content-Type': "application/json"}
    # Post requests
    res = requests.post(url,data=payload,headers=headers,params=params)
        
    # Check requests
    try:
        res.raise_for_status()
    except requests.HTTPError as e:
        raise (e)
    res_json = res.json()

    # Fetch the task id of the current task created
    task_id = res_json.get("engagement").get("id")
    print("🎆 Task created successfully: ", task_id)
        
    return res_json
```


```python
TASK_ID_LIST = []
for i in HS_CONTACT_ID_LIST:
    contact_id = i[0]
    contact = hs.contacts.get(contact_id)
    contact_props = contact.get('properties')
    contact_linkedin_url = i[1]
    hs_contact_url = HS_CONTACT_URL + contact_id
    #print(hs_contact_url)
    result = create_task(owner_id=HS_OWNER_ID, 
                tstamp=tstamp, 
                contact_id=contact_id ,
                contact_props = contact_props,
                contact_linkedin_url=contact_linkedin_url,
                hs_contact_url = hs_contact_url, 
                engagement="TASK")
    TASK_ID_LIST.append(result.get("engagement").get("id"))
```


```python
if not TASK_ID_LIST:
    print("No tasks created")
else:
    email_to = SEND_EMAIL_TO #to send the report
    subject = "LinkedIN Leads Alert"
    content = email_body

    naas.notification.send(email_to=email_to, subject=subject, html=content)
```


```python

```
