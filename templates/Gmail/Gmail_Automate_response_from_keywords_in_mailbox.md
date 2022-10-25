<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Gmail/Gmail_Automate_response_from_keywords_in_mailbox.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Gmail+-+Automate+response+from+keywords+in+mailbox:+Error+short+description">🚨 Bug report</a>

**Tags:** #gmail #productivity #naas_drivers #operations #snippet

**Author:** [Sanjay Sabu](https://www.linkedin.com/in/sanjay-sabu-4205/)

## Input

### Import librairies


```python
import naas
from naas_drivers import email
from re import search
```

### Account credentials


```python
username = "**********@gmail.com"
to = "**********@gmail.com"
password = "**********"
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
dataframe = emails.get(criteria="ALL")
df
```

## Output

### Automated reponse


```python
for df in dataframe["text"]:
    text = df.lower()
    if search("sales report", text): 
        email_to = "naas.sanjay22@gmail.com"
        subject = "Sales Report"
        content = "Hi \n,Here I am attaching the sales report as per your request\n.With Regards\n,NAAS Team"
        files = ["Excel-Sales_Feb2020.csv"]
        naas.notifications.send(email_to=email_to, subject=subject, html=content, files=files)
```
