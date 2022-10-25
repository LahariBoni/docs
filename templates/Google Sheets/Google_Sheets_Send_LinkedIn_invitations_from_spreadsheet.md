<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Sheets/Google_Sheets_Send_LinkedIn_invitations_from_spreadsheet.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Sheets+-+Send+LinkedIn+invitations+from+spreadsheet:+Error+short+description">🚨 Bug report</a>

**Tags:** #googlesheets #invitation #automation #content #notification #email #linkedin 

**Author:** [Valentin Goulet](https://www.linkedin.com/in/valentin-goulet-3a3070152/)

## Input

### Import libraries


```python
import naas
from naas_drivers import linkedin, gsheet
```

### Setup Google Sheets
👉 The spreadsheet needs to contains profil's url on the 1st column of the file.


```python
spreadsheet_id = "YOUR_SPREADSHEET_ID"
sheet_name = "YOUR_SHEET_NAME"
profile_col_name = "url"
```

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn cookies
LI_AT = 'YOUR_COOKIE_LI_AT'
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'

# LinkedIn limit invitations up to 100 per week (Becareful !)
add_per_launch = 4 
```

### Schedule your notebook


```python
# Scheduler your invitation everyday at 8:00 AM
naas.scheduler.add(cron="0 8 * * *")

# Uncomment the line below to delete your scheduler
# naas.scheduler.delete()
```

## Model

### Get all list of profiles


```python
df = gsheet.connect(spreadsheet_id).get(sheet_name=sheet_name)
```

### Get restricted list


```python
# Alert when last than 20 urls remains in the gsheet
if len(df) < 20:
    email_to = "YOUR_EMAIL"
    subject = "Invite LinkedIn alert : " + str(len(df)) + " lines left in the Linkedin's url database"
    content = "You can add more lines to the gsheet or update the Notebook to set a new spreadsheet !"
    naas.notification.send(email_to=email_to, subject=subject, html=content)

df = df.head(add_per_launch)
print("Invits will be send to :")
df
```

## Output

### Send the invitation and delete profil from Gsheet


```python
for index, row in df.iterrows():
    profile = row[profile_col_name]
    result = linkedin.connect(LI_AT, JSESSIONID).invitation.send(recipient_url=profile)
    print(f"Invitation sent to : {profile}")
    # Suppression de la ligne du gsheet
    gsheet.connect(spreadsheet_id).delete(sheet_name=sheet_name, rows=[2])
```
