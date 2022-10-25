<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Gmail/Gmail_Clean_mailbox.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Gmail+-+Clean+mailbox:+Error+short+description">🚨 Bug report</a>

**Tags:** #gmail #productivity #naas_drivers #operations #analytics #plotly

**Author:** [Martin Donadieu](https://www.linkedin.com/in/martindonadieu)

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
username = "**********@gmail.com"
password = "**********"
smtp_server = "imap.gmail.com"
box = "INBOX"
```

Note: You need to create an application password following this procedure - https://support.google.com/mail/answer/185833?hl=en

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
dataframe = emails.get(criteria="ALL")
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
indexes = np.unique(sender_name, return_index=True)[1]
[sender_name[index] for index in sorted(indexes)]

indexes = np.unique(sender_email, return_index=True)[1]
[sender_email[index] for index in sorted(indexes)]
total_email = len(emails.get(criteria="ALL"))
c = 0
for i in sender_email:
    new_row = {'SENDER_NAME':sender_name[c],'SENDER_EMAIL':i,'COUNT':sender_email.count(i),'PERCENTAGE':round(((sender_email.count(i))/total_email)*100)}
    result = result.append(new_row, ignore_index=True)
    c+=1
result = result.drop_duplicates()
result.sort_values(by=['COUNT'], inplace=True, ascending=False)
result
```

### Email graph plot


```python
fig = px.bar(x=result['COUNT'], y=result['SENDER_EMAIL'], orientation='h')
fig.show()
```

### Deleting using uid


```python
%%time
uid = [21]   #uid of mails to be deleted
uid = map(str, uid)  
flag = "DELETED"
for i in uid:
    attachments = emails.set_flag(i, flag, True)
```

## Output

### Deleting using email id


```python
d_email = "notifications@naas.ai"  # email id to be deleted
data_from = dataframe['from']
data_uid = dataframe['uid']
uid = []
for i in range(len(dataframe)):
    if data_from[i]['email'] == d_email:
        uid.append(data_uid[i])
```

### Deleting the emails


```python
for i in uid:
    attachments = emails.set_flag(i, "DELETED", True)
```

### Showing the upated email list


```python
dataframe = emails.get(criteria="ALL")
dataframe
```
