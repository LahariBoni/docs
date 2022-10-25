<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Create_contacts_from_linkedin_post_likes.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Create+contacts+from+linkedin+post+likes:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #contact #naas_drivers #linkedin #post #contact #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import librairies


```python
from naas_drivers import linkedin, hubspot
import naas
import requests
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_KEY = 'YOUR_HUBSPOT_API_KEY'
```

### Setup your LinkedIn

<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Coookies
LI_AT = "AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2" # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = "ajax:837990740022038xxxxx" # EXAMPLE ajax:8379907400220387585

# Post URL
POST_URL = "----"
```

## Model

### Get post likes


```python
df_posts = linkedin.connect(LI_AT, JSESSIONID).post.get_likes(POST_URL)

# Display the number of likes
print("Number of likes: ", df_posts.PROFILE_URN.count())
```


```python
# Show dataframe with list of profiles from likes 
df_posts
```

### Create contacts from LinkedIn post likes


```python
def create_contacts_from_post(df,
                              c_profile_urn="PROFILE_URN",
                              c_firstname="FIRSTNAME",
                              c_lastname="LASTNAME",
                              c_occupation="OCCUPATION"):
    for _, row in df.iterrows():
        profile_urn = row[c_profile_urn]
        firstname = row[c_firstname]
        lastname = row[c_lastname]
        occupation = row[c_occupation]
        linkedinbio = f"https://www.linkedin.com/in/{profile_urn}"
        email = None
        phone = None

        # contact
        try:
            contact = linkedin.connect(LI_AT, JSESSIONID).profile.get_contact(linkedinbio)
            email = contact.loc[0, "EMAIL"]
            phone = contact.loc[0, "PHONENUMBER"]
        except:
            print("No contact info")

        # With send method
        data = {"properties": 
                {
                    "linkedinbio": linkedinbio,
                    "firstname": firstname,
                    "lastname": lastname,
                    "jobtitle": occupation,
                    "email": email,
                    "phone": phone,
                }
               }
        print(data)
        hubspot.connect(HS_API_KEY).contacts.send(data)
```

## Output

### Display result


```python
create_contacts_from_post(df_posts)
```
