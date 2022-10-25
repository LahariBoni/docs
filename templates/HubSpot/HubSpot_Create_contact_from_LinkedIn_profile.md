<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Create_contact_from_LinkedIn_profile.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Create+contact+from+LinkedIn+profile:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #linkedin #profile #naas_drivers #snippet #sales

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook creates a contact in HubSpot from a LinkedIn profile URL with:
- email
- linkedinbio
- phone and mobilephone
- website
- twitterhandle
- firstname
- lastname
- info
- jobtitle
- industry
- city
- state
- country
- job_function
- company
- field_of_study

## Input

### Import libraries


```python
import naas
from naas_drivers import hubspot, linkedin
from os import path, mkdir
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
# HubSpot API Key
HS_API_KEY = naas.secret.get("HS_API_KEY") or 'YOUR_HUBSPOT_API_KEY'

# Owner ID -> Go to Contact -> Manage properties -> Contact owner to find your owner ID
HS_OWNER_ID = "158373005"
```

### Setup LinkedIn
If you are using the Chrome Extension:

- [Install Naas Chrome Extension](https://chrome.google.com/webstore/detail/naas/cpkgfedlkfiknjpkmhcglmjiefnechpp?hl=fr&authuser=0)
- [Create a new token](https://app.naas.ai/hub/token)
- Copy/Paste your token in your extension
- Login/Logout your LinkedIn account
- Your secrets "LINKEDIN_LI_AT" and "LINKEDIN_JSESSIONID" will be added directly on your naas everytime you login and logout.

or <br>

If you are not using the Google Chrome Extension, [learn how to get your cookies on LinkedIn](https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75) and set up the values below:
- 🍪 li_at
- 🍪 JSESSIONID


```python
# Cookies
LI_AT = naas.secret.get("LINKEDIN_LI_AT") or 'YOUR_COOKIE_LI_AT'
JSESSIONID = naas.secret.get("LINKEDIN_JSESSIONID") or 'YOUR_COOKIE_JSESSIONID'

# LinkedIn profile to be created
LINKEDIN_PROFILE_URL = "https://www.linkedin.com/in/xxxxxxxxxx/"

# Directory to stored data extracted from LinkedIn
# By default data will be stored in a folder named with the LinkedIn public ID on the same folder of your notebook
DIR_PATH = None #EXAMPLE: 'outputs/contact-name'
```

## Model

### Setup outputs path if needed


```python
if DIR_PATH is None:
    DIR_PATH = LINKEDIN_PROFILE_URL.split("/in/")[-1].split("/")[0]
    print("-> Data will be stored in folder:", DIR_PATH)
```

### Create directory to store result


```python
def create_dir(dir_path):
    if not path.exists(dir_path):
        mkdir(dir_path)
```

### Get LinkedIn info


```python
def get_linkedin_info(url, info, dir_path=None):
    if info == "identity":
        df = linkedin.connect(LI_AT, JSESSIONID).profile.get_identity(url)
    elif info == "network":
        df = linkedin.connect(LI_AT, JSESSIONID).profile.get_network(url)
    elif info == "contact":
        df = linkedin.connect(LI_AT, JSESSIONID).profile.get_contact(url)
    elif info == "resume":
        df = linkedin.connect(LI_AT, JSESSIONID).profile.get_resume(url)
    elif info == "company":
        df = linkedin.connect(LI_AT, JSESSIONID).company.get_info(url)
        
    if len(df) > 0:
        if "PROFILE_ID" in df.columns:
            linkedin_id = df.loc[0, "PROFILE_ID"]
        elif "COMPANY_ID" in df.columns:
            linkedin_id = df.loc[0, "COMPANY_ID"]
        # Setup directory path
        if dir_path is None:
            dir_path = linkedin_id
            
        # Create directory
        create_dir(dir_path)
        
        # File path
        file_path = f"LINKEDIN_{info.upper()}_{linkedin_id}.csv"
        
        # Save dataframe in csv
        df.to_csv(path.join(dir_path, file_path), index=False)
    return df
```

### Create contact in HubSpot


```python
def create_hubspot_contact(df, properties={}):
    if len(df) > 0:
        # Init variables
        email = str(df.iloc[0]['EMAIL'])
        public_id = str(df.iloc[0]['PUBLIC_ID'])
        profile_id = str(df.iloc[0]['PROFILE_ID'])
        linkedinbio = str(df.iloc[0]['PROFILE_URL'])
        phone = str(df.iloc[0]['PHONENUMBER'])
        website = str(df.iloc[0]['WEBSITES'])
        twitterhandle = str(df.iloc[0]['TWITTER'])
        
        # If the email is not in LinkedIn profile then populate LINKEDIN_ID@TBD.com
        if email == "None":
            email = f"{public_id}@unknown-email.com"
        
        # Update json properties
        properties["email"] = email
        properties["linkedinbio"] = linkedinbio
        properties["phone"] = phone
        properties["mobilephone"] = phone
        properties["website"] = website
        properties["twitterhandle"] = twitterhandle
        
    contact_id = hubspot.connect(HS_API_KEY).contacts.send({"properties": properties})
    print(f"✅ Contact {email} created in HubSpot: {properties}")
    return properties, contact_id, public_id
```

### Update Contact Owner in HubSpot


```python
def update_hubspot_owner(owner_id, hs_object_id, hubspot_owner_id="", properties={}):
    if str(hubspot_owner_id) != owner_id:
        properties = {"hubspot_owner_id": owner_id}
        hubspot.connect(HS_API_KEY).contacts.patch(hs_object_id, {"properties": properties})
        print(f"✅ Contact owner updated in HubSpot: {owner_id}")
    else:
        print(f"👉 Contact owner already set in HubSpot: {owner_id}")
    return properties
```

### Update LinkedIn identity in HubSpot


```python
def update_hubspot_lk_identity(df, hs_object_id, properties={}):
    if len(df) > 0:
        # Init variables
        firstname = str(df.iloc[0]['FIRSTNAME'])
        lastname = str(df.iloc[0]['LASTNAME'])
        info = str(df.iloc[0]['SUMMARY'])
        jobtitle = str(df.iloc[0]['OCCUPATION'])
        industry = str(df.iloc[0]['INDUSTRY_NAME'])
        city = "None"
        state = "None"
        region = str(df.iloc[0]['REGION'])
        if region != "None":
            r = region.split(",")
            if len(r) > 1:
                city = region.split(",")[0].strip()
                state = region.split(",")[1].strip()
            else:
                state = region
        country = str(df.iloc[0]['COUNTRY'])
        
        # Update json properties
        properties["firstname"] = firstname
        properties["lastname"] = lastname
        properties["info"] = info
        properties["jobtitle"] = jobtitle
        properties["industry"] = industry
        properties["city"] = city
        properties["state"] = state
        properties["country"] = country
        
    hubspot.connect(HS_API_KEY).contacts.patch(hs_object_id, {"properties": properties})
    print(f"✅ Contact identity updated in HubSpot: {properties}")
    return properties
```

### Update LinkedIn network in HubSpot


```python
def update_hubspot_lk_network(df, hs_object_id, properties={}):
    if len(df) > 0:
        # Init variables
        linkedinconnections = df.iloc[0]['FOLLOWERS_COUNT']
        linkedin_distance = df.iloc[0]['DISTANCE']
        
        # Update json properties
        properties["linkedinconnections"] = linkedinconnections
        properties["linkedin_distance"] = linkedin_distance
        
    hubspot.connect(HS_API_KEY).contacts.patch(hs_object_id, {"properties": properties})
    print(f"✅ Contact network updated in HubSpot: {properties}")
    return properties
```

### Update LinkedIn resume in HubSpot


```python
def update_hubspot_lk_resume(df, hs_object_id, dir_path=None, properties={}):
    if len(df) > 0:
        df_exp = df[df["CATEGORY"] == "Experience"].reset_index(drop=True)
        if len(df_exp) > 0:
            # Job function
            properties["job_function"] = str(df_exp.iloc[0]['TITLE'])
            # Company
            properties["company"] = str(df_exp.iloc[0]['PLACE'])
            company_id = str(df_exp.iloc[0]['PLACE_ID'])
            get_linkedin_info(company_id, "company", dir_path=dir_path)

        df_study = df[df["CATEGORY"] == "Education"].reset_index(drop=True)
        if len(df_study) > 0:
            # Field of study
            properties["field_of_study"] = str(df_study.iloc[0]['FIELD'])
        
    hubspot.connect(HS_API_KEY).contacts.patch(hs_object_id, {"properties": properties})
    print(f"✅ Contact resume updated in HubSpot: {properties}")
    return properties
```

## Output

### Update contact in HubSpot


```python
def create_contact_husbpot(linkedin_url, owner_id, dir_path=None):
    # Init properties
    properties = {}

    # Update LinkedIn contact in HubSpot
    df_contact = get_linkedin_info(linkedin_url, "contact", dir_path)
    p_c, hs_object_id, email = create_hubspot_contact(df_contact)
    dir_path = email
    properties.update(p_c)

    # Update Contact Owner in HubSpot
    p_o = update_hubspot_owner(owner_id, hs_object_id)
    properties.update(p_o)

    # Update LinkedIn identity in HubSpot
    df_identity = get_linkedin_info(linkedin_url, "identity", dir_path)
    p_i = update_hubspot_lk_identity(df_identity, hs_object_id)
    properties.update(p_i)

    # Update LinkedIn network in HubSpot
    df_network = get_linkedin_info(linkedin_url, "network", dir_path)
    p_n = update_hubspot_lk_network(df_network, hs_object_id)
    properties.update(p_n)

    # Update LinkedIn resume in HubSpot
    df_resume = get_linkedin_info(linkedin_url, "resume", dir_path)
    p_r = update_hubspot_lk_resume(df_resume, hs_object_id, dir_path)
    properties.update(p_r)
    print(f"\n✅ Contact successfully created in HubSpot: {linkedin_url}", properties)
        
create_contact_husbpot(LINKEDIN_PROFILE_URL, HS_OWNER_ID, DIR_PATH)
```
