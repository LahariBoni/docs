<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Sheets/Google_Sheets_Send_emails_from_sheet.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Sheets+-+Send+emails+from+sheet:+Error+short+description">🚨 Bug report</a>

**Tags:** #googlesheets #gsheet #data #naas_drivers #operations #snippet #email

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import librairies


```python
from naas_drivers import gsheet
from naas_drivers import email
```

### Variables


```python
username = "USERNAME"
password = "PASSWORD"
email_from = "***@cashstory.com",
smtp_server = "smtp.gmail.com",
smtp_port = 465,
smtp_type = "SSL",
```

## Model

### Connect to your gmail account


```python
gmail = emails.connect(username,
                       password,
                       email_from,
                       smtp_server,
                       smtp_port,
                       smtp_type)
```

### Get email list from Gsheet


```python
spreadsheet_id = "1s-TQZrevbmveFKlx2H49fgvr_nZPEY_ffoi0iWal**E"
sheet_name = "********"

df = gsheet.connect(spreadsheet_id).get(sheet_name)
df
```


```python
emails = df['EMAIL'].drop_duplicates().values
print(emails)
```

## Output

### Send emails batchs


```python
subject = "The tesla action is going up"
content = "check in the link the chart data maide from fresh dataset : [LINK]"

for email in emails:
    print(email)
#     gmail.send(email_to=email, subject=subject, html=content)
```
