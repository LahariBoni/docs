<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Send_connections_from_network_to_gsheet.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Send+connections+from+network+to+gsheet:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #network #connections #naas_drivers #csv #automation #content #googlesheets

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
from naas_drivers import linkedin, gsheet
import naas
import pandas as pd
```

### Setup LinkedIn
👉 <a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# Lindekin cookies
LI_AT = "AQEDARCNSioDe6wmAAABfqF-HR4AAAF-xYqhHlYAtSu7EZZEpFer0UZF-GLuz2DNSz4asOOyCRxPGFjenv37irMObYYgxxxxxxx"
JSESSIONID = "ajax:12XXXXXXXXXXXXXXXXX"
```

### Setup your Google Sheet
👉 Get your spreadsheet URL<br>
👉 Share your gsheet with our service account to connect : naas-share@naas-gsheets.iam.gserviceaccount.com<br>
👉 Create your sheet before sending data into it


```python
# Spreadsheet URL
SPREADSHEET_URL = "https://docs.google.com/spreadsheets/d/XXXXXXXXXXXXXXXXXXXX"

# Sheet name
SHEET_NAME = "LK_CONNECTIONS"
```

### Setup Naas


```python
naas.scheduler.add(cron="0 8 * * *")

#-> To delete your scheduler, please uncomment the line below and execute this cell
# naas.scheduler.delete()
```

## Model

### Get connections from Google Sheet


```python
df_gsheet = gsheet.connect(SPREADSHEET_URL).get(sheet_name=SHEET_NAME)
df_gsheet
```

### Get new connections


```python
def get_new_connections(df_gsheet, key="PROFILE_URN"):
    profiles = []
    if len(df_gsheet) > 0:
        profiles = df_gsheet[key].unique()
    else:
        df = linkedin.connect(LI_AT, JSESSIONID).network.get_connections(limit=-1)
        return df
    
    # Get new
    df_new = pd.DataFrame()
    update = True
    while update:
        start = 0
        df = linkedin.connect(LI_AT, JSESSIONID).network.get_connections(start=start, count=100, limit=100)
        new_profiles = df[key].unique()
        for i, p in enumerate(new_profiles):
            if p in profiles:
                update = False
                df = df[:i]
                break
        start += 100
        df_new = pd.concat([df_new, df])
    return df_new

df_new = get_new_connections(df_gsheet, key="PROFILE_URN")
df_new
```

## Output

### Send to Google Sheet


```python
gsheet.connect(SPREADSHEET_URL).send(df_new,
                                     sheet_name=SHEET_NAME,
                                     append=True)
```
