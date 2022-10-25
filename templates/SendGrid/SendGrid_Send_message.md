<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/SendGrid/SendGrid_Send_message.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=SendGrid+-+Send+message:+Error+short+description">🚨 Bug report</a>

**Tags:** #sendgrid #message #snippet #operations

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)

This notebook enables you to send email with SendGrid

## Input

### Imports


```python
import requests
from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import *
```

### Setup SendGrid
👉 Get your [api key](https://app.sendgrid.com/settings/api_keys)


```python
SENDGRID_API_KEY = ""
```

### Setup your parameters


```python
# Email senders and receivers
email_from ="" 
email_from_name = None #custom email from name, 24 char max
email_to = ""
email_to_name = None #custom email from name
email_to_cc = None #emails in copy

# Email content
subject="Test SendGrid"
content="Hi there, Best regards!"
```

## Model

### Sending mails via sendgrid


```python
def send_email(email_from,
               email_to,
               subject,
               content,
               email_to_cc=None,
               email_from_name=None,
               email_to_name=None,
               content_type="text/plain"):
    
    # Connect to SendGrid
    SG = SendGridAPIClient(api_key=SENDGRID_API_KEY)

    from_email = Email(email_from, name=email_from_name)
    to_email = To(email_to, name=email_to_name)
    cc_email = Cc(email=email_to_cc)
    content = Content(content_type, content)

    mail = Mail(from_email, to_email, subject, content)
    if email_to_cc is not None:
        mail.add_cc(cc_email)
    try:
        res = SG.client.mail.send.post(request_body=mail.get())
        if res.status_code == 202:
            print(f"📧 Email successfully sent to {email_to}")
    except requests.HTTPError as e:
        raise(e)
```

## Output

### Send the mail


```python
send_email(email_from=email_from,
           email_to=email_to,
           subject=subject,
           content=content,
           email_to_cc=email_to_cc,
           email_from_name=email_from_name,
           email_to_name=email_to_name)
```
