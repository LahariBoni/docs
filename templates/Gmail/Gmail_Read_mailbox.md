<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Gmail/Gmail_Read_mailbox.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Gmail+-+Read+mailbox:+Error+short+description">🚨 Bug report</a>

**Tags:** #gmail #productivity #naas_drivers #operations #snippet #dataframe

**Author:** [Martin Donadieu](https://www.linkedin.com/in/martindonadieu)

## Input

### Import librairy


```python
from naas_drivers import email
```

### Account credentials


```python
username = "*********@gmail.com"
to = "*********@gmail.com"
password = "*********"
smtp_server = "imap.gmail.com"
box = "INBOX"
```

## Model

### Connect to email box


```python
emails = email.connect(username, 
        password, 
        username, 
        smtp_server)
```

### Get email list


```python
df = emails.get(criteria="unseen")
df
```


```python
uid_list = df['uid'].tolist() 
uid_list
```

### Setup the flags


```python
%%time
uid = uid_list
flag = "DELETED"
# possible value for flag:
# flag = 'SEEN'
# flag = 'ANSWERED'
# flag = 'FLAGGED'
# flag = 'DELETED'
# flag = 'DRAFT'
# flag = 'RECENT'
```

## Output

### Read mailbox


```python
attachments = emails.set_flag(uid, flag, True)
```
