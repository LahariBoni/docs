<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Gmail/Gmail_Schedule_mailbox_cleaning.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Gmail+-+Schedule+mailbox+cleaning:+Error+short+description">🚨 Bug report</a>

**Tags:** #gmail #productivity #naas_drivers #operations #automation #analytics #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import libraries


```python
import naas
from naas_drivers import email
import pandas as pd
import numpy as np
import plotly.express as px
```

### Account credentials


```python
username = "naas.sanjay22@gmail.com"
password = "atsuwkylwfhucugw"
smtp_server = "imap.gmail.com"
box = "INBOX"
```

Note: You need to create an application password following this procedure - https://support.google.com/mail/answer/185833?hl=en

### Setting the scheduler


```python
naas.scheduler.add(recurrence="0 9 * * *") # Scheduler set for 9 am
```

## Model

### Connect to email box


```python
emails = naas_drivers.email.connect(username, 
        password, 
        username, 
        smtp_server)
```

### Get email list


```python
dataframe = emails.get(criteria="seen")
dataframe
```

### Creating dataframe and inserting values


```python
sender_name = []
sender_email = []
for df in dataframe["from"]:
    sender_name.append(df['name'])
    sender_email.append(df['email'])
result = pd.DataFrame(columns = ['SENDER_NAME','SENDER_EMAIL','COUNT','PERCENTAGE'])
name_unique = np.unique(sender_name)
email_unique = np.unique(sender_email)
total_email = len(emails.get(criteria="seen")) + len(emails.get(criteria="unseen"))
c = 0
for i in np.unique(sender_name):
    new_row = {'SENDER_NAME':i,'SENDER_EMAIL':sender_email[c],'COUNT':sender_name.count(i),'PERCENTAGE':round(((sender_name.count(i))/total_email)*100)}
    result = result.append(new_row, ignore_index=True)
    c+=1
result
```

### Email graph plot


```python
fig = px.bar(x=result['COUNT'], y=result['SENDER_NAME'], orientation='h')
fig.show()
```

## Output

### Deleting using email id


```python
d_email = "notifications@naas.ai"  # email id to be deleted
data_from = dataframe['from']
data_uid = dataframe['uid']
uid = []
```

### Updating the uid values


```python
for i in range(len(dataframe)):
    if data_from[i]['email'] == d_email:
        uid.append(data_uid[i])
print(uid)
```

### Deleting the emails


```python
for i in uid:
    attachments = emails.set_flag(i, "DELETED", True)
```

### Showing the upated email list


```python
dataframe = emails.get(criteria="seen")
dataframe
```
