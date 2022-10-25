<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_company_followers.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+company+followers:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #company #followers #naas_drivers #automation #csv #content

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This templates will get all followers from your LinkedIn company and save it into a CSV.<br><br>
**Available columns :**
- FIRSTNAME : First name
- LASTNAME : Last name
- OCCUPATION : Text below the name in the profile page
- PROFILE_PICTURE : Profile picture URL
- PROFILE_URL : Profile URL
- PROFILE_ID : LinkedIn profile id
- PUBLIC_ID : LinkedIn public profile id
- FOLLOWED_AT : Date of following company
- DISTANCE : Distance between your profile

*NB: Due to LinkedIn restriction, during the first execution, the follow at date will correspond to the 15 of the month.<br>
Then, the exact date will be retrieved for every new followers.*

## Input

### Import library


```python
from naas_drivers import linkedin
import pandas as pd
import naas
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Credentials
LI_AT = None or naas.secret.get("LI_AT")  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = None or naas.secret.get("JSESSIONID")  # EXAMPLE ajax:8379907400220387585

# Company URL
COMPANY_URL = "https://www.linkedin.com/company/naas-ai/"
```

### Setup Variables


```python
# Inputs
company_name = COMPANY_URL.strip().split("company/")[-1].split("/")[0]
csv_input = f"LINKEDIN_COMPANY_FOLLOWERS_{company_name}.csv"
```

### Setup Naas scheduler


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

### Update followers


```python
def update_connections(df, file_path, key=None):
    # Init output
    df_update = pd.DataFrame()
    
    # Init df posts is empty then return entire database
    if len(df) > 0:
        # If dataframe not empty, get last connections
        profiles = df[key].unique()
        start = 0
        count = 1
        while True:
            tmp_new = linkedin.connect(LI_AT, JSESSIONID).company.get_followers(start=start, count=count, limit=count)
            # Check if existing profile in each call
            tmp_new = tmp_new[~tmp_new.PROFILE_ID.isin(profiles)]
            df_update = pd.concat([df_update, tmp_new])
            if len(tmp_new) == 0:
                break
            
            # Get more profile
            start += count
        print(f"-> New followers fetched: {len(df_update)}.")
    else:
        df_update = linkedin.connect(LI_AT, JSESSIONID).company.get_followers(count=100, limit=-1)
        
    # Concat data
    df = pd.concat([df_update, df]).drop_duplicates(key, keep="first")
    return df.reset_index(drop=True)
    
df_update = update_connections(df_followers, csv_input, key="PROFILE_ID")
df_update
```

## Output

### Save dataframe in CSV


```python
df_update.to_csv(csv_input, index=False)
```

### Add dependency to production


```python
naas.dependency.add(csv_input)
```
