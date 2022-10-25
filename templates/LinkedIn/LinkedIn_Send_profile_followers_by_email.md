<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Send_profile_followers_by_email.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Send+profile+followers+by+email:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #network #followers #naas_drivers #content #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This templates will get all followers from LinkedIn profile and send CSV by email.<br><br>
**Available columns :**
- PROFILE_ID: LinkedIn profile id
- PROFILE_URL: Profile URL
- PUBLIC_ID: LinkedIn public profile id
- FIRSTNAME: First name
- LASTNAME: Last name
- FULLNAME: First name + Last name
- OCCUPATION: Text below the name in the profile page
- PROFILE_PICTURE: Profile picture URL
- BACKGROUND_PICTURE: Background profile picture URL
- FOLLOWER_COUNT: No de followers
- FOLLOWING: True if you are following back, False or Unknown
- INFLUENCER: True, False or Unknown
Note : Due to follower privacy, it will not match with the exact number of followers.<br>

## Input

### Import library


```python
from naas_drivers import linkedin, emailbuilder
import naas
import pandas as pd
from datetime import datetime
```

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn cookies
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585
```

### Setup Outputs
Create CSV to store your followers.<br>
PS: This CSV could be used in others LinkedIn templates.


```python
csv_output = f"LINKEDIN_FOLLOWERS.csv"
```

### Setup Naas notification


```python
EMAIL_TO = "ENTER_RECIPIENT_EMAIL_HERE" # you will receive weekly summary at this email 
EMAIL_FROM = "ENTER_SENDER_EMAIL_HERE" # summary will have this email as sender. Only available for your naas email
EMAIL_SUBJECT = 'LinkedIn Followers Export' # subject of your email
```

### Setup Naas scheduler
Schedule your notebook with the naas scheduler feature


```python
# the default settings below will make the notebook run at 08:00 on the 1st of every month
# for information on changing this setting, please check https://crontab.guru/ for information on the required CRON syntax 
naas.scheduler.add(cron="0 8 1 * *")

# to de-schedule this notebook, simply run the following command: 
# naas.scheduler.delete()
```

## Model

### Get followers from CSV


```python
def read_csv(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_followers = read_csv(csv_output)
df_followers
```

### Update new followers
If CSV is empty, we will get all your followers


```python
def update_last_posts(df, csv_output):
    # Init output
    df_new = pd.DataFrame()
    
    # Check if dataframe init is empty
    if len(df) > 0:
        # If dataframe not empty, get last followers
        profiles = df.PROFILE_ID.unique()
        start = 0
        count = 100
        while True:
            tmp_new = linkedin.connect(LI_AT, JSESSIONID).network.get_followers(start=start, limit=count)
            # Check if existing profile in each call
            tmp_df = tmp_new[tmp_new.PROFILE_ID.isin(profiles)]
            if len(tmp_df) > 0:
                break
            
            # Get more profile
            df_new = pd.concat([df_new, tmp_new])
            start += count
    else:
        # If dataframe empty, get all followers
        df_new = linkedin.connect(LI_AT, JSESSIONID).network.get_followers(limit=-1)
        
    # Concat
    print(f"-> New followers fetched: {len(df_new)}.")
    df_update = pd.concat([df, df_new])
    
    # Cleaning to remove duplicates if needed
    if len(df_update) > 0:
        df_update= df_update.drop_duplicates("PROFILE_ID", keep="first")
        
    # Add extract date
    df_update["DATE_EXTRACT"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
    # Save dataframe in CSV
    df_update.to_csv(csv_output, index=False)
    
    # Return all followers
    print(f"Total followers fetched: {len(df_update)}.")
    return df_update.reset_index(drop=True)
    
df_update = update_last_posts(df_followers, csv_output)
df_update
```

### Share output with naas


```python
# Share output with naas
csv_link = naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(csv_output)
```

### Create email content


```python
def email_content(csv_link):
    content = {
       "header_naas": ("<a href='https://www.naas.ai/'>"
                       "<img align='center' width='30%' target='_blank' style='border-radius:5px;'"
                       "src='https://landen.imgix.net/jtci2pxwjczr/assets/5ice39g4.png?w=160'"
                       "alt='Please provide more information.'/>"
                       "</a>"),
        "txt_0": emailbuilder.text(f"Hi there,<br><br>"
                                   f"Here's your LinkedIn followers report as of {datetime.now().strftime('%Y-%m-%d')}.<br><br>"),
        "button": emailbuilder.button(csv_link, "Download CSV"),
        "signature": "Naas Team",
        "footer": emailbuilder.footer_company(naas=True)
    }
    email_content = emailbuilder.generate(display='iframe', **content)
    return email_content

email_content = email_content(csv_link)
```

## Output

### Send followers by email


```python
# sends the email
naas.notification.send(email_to=EMAIL_TO,
                       subject=EMAIL_SUBJECT,
                       html=email_content,
                       email_from=EMAIL_FROM,
                       files=[csv_output])
```
