<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Slack/Slack_Add_new_user_to_Google_Sheets.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Slack+-+Add+new+user+to+Google+Sheets:+Error+short+description">🚨 Bug report</a>

**Tags:** #slack #googlesheets #operations #automation


**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)

With this notebook, new Slack users are automatically added to your Google Sheets.
<br/>References :
- Slack api :
- Slack SDK to use : [https://github.com/slackapi/python-slack-sdk](https://github.com/slackapi/python-slack-sdk)
- Google Sheet naas driver : [https://docs.naas.ai/templates/google-sheets](https://docs.naas.ai/templates/google-sheets)


## Input


### Import libraries



```python
!pip install slack-sdk --user
```


```python
from naas_drivers import gsheet
import naas
import pandas as pd
from slack_sdk import WebClient
from datetime import datetime
```

### Setup Slack



```python
SLACK_BOT_TOKEN = "xoxb-232887839156-1673274923699-vTF6xxxxxxxxxx"
```


```python
client = WebClient(token=SLACK_BOT_TOKEN)
```

### Setup Google Sheets

- Share your sheet with our service account : 🔗 [naas-share@naas-gsheets.iam.gserviceaccount.com](mailto:naas-share@naas-gsheets.iam.gserviceaccount.com)



```python
SPREADSHEET_URL = "---"
SHEET_NAME = "Sheet1"
```

### Setup Naas



```python
# Schedule your notebook every hour
naas.scheduler.add(cron="0 * * * *")

#-> Uncomment the line below and execute this cell to delete your scheduler
# naas.scheduler.delete()
```

## Model


### List users from Slack workspace



```python
def list_users():
    df = pd.DataFrame()
    idx=0
    for user_data in client.users_list().data['members']:
        if ('real_name' in user_data and user_data['real_name'] != 'Slackbot') and not user_data['is_bot']:
            df.loc[idx,'NAME'] = user_data['profile']['real_name']
            df.loc[idx,'ID'] = user_data['id']
            df.loc[idx,'FIRST_VIEWED_AT'] = datetime.fromtimestamp(user_data['updated'])
            idx+=1
    
    return df

df_slack = list_users()
df_slack
```

### Get users from Google Sheet



```python
df_gsheet = gsheet.connect(SPREADSHEET_URL).get(sheet_name=SHEET_NAME)
df_gsheet
```

### Get new users in Slack



```python
def get_new_users(df_slack, df_gsheet):
    if len(df_gsheet) == 0:
        return df_slack
    else:
        historical_data = df_gsheet.ID.to_list()
        df_new=pd.DataFrame()
        for idx, row in df_slack.iterrows():
            if row['ID'] not in historical_data:
                df_new = df_new.append(row)
        return df_new

df_new = get_new_users(df_slack, df_gsheet)
df_new
```

## Output


### Send new users to Google Sheets



```python
# Send data to Google Sheets
gsheet.connect(SPREADSHEET_URL).send(
    sheet_name=SHEET_NAME,
    data=df_new,
	append=True
)
```
