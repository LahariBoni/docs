<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Webhook_demo.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Webhook+demo:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #webhook #snippet #operations

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Basic formulas
Running this command will add your this notebook to the "⚡️ Production" folder. <br>
You can then, trigger it with the generated URL.


```python
import naas 
naas.webhook.add()
```

Running this command will delete your webhook


```python
naas.webhook.delete()
```

## How to use webhook with 3rd party tool?
Use the webhook feature to get data and trigger action from any tool.<br>
Example of UMSO - Nocode website builder : https://www.umso.com/

## Input

### Import library


```python
import naas
```

### Variables
Add "parameters" tag in cells => Code will be injected after this cell


```python
# Parameters
params = {}
body = {
    "Domain": "www.naas.ai",
    "Email": "jeremy.naas42@gmail.com",
    "Name": "Jeremy",
    "Site_ID": "jtci2pxwjczr",
    "Submitted_At": "March 20, 2021 16:55 UTC",
}
headers = {
    "host": "public.naas.ai",
    "x-request-id": "e6b994eb9a83b9cac794c0d9e57c1533",
    "x-real-ip": "10.0.87.40",
    "x-forwarded-for": "10.0.87.40",
    "x-forwarded-host": "public.naas.ai",
    "x-forwarded-port": "443",
    "x-forwarded-proto": "https",
    "x-scheme": "https",
    "accept": "application/json",
    "content-type": "application/json",
    "user-agent": "landen-webhook-agent",
    "accept-encoding": "gzip",
    "connection": "close",
    "content-length": "142",
    "accept-charset": "utf-8",
}
```

## Model

### Create webhook, get url, add it to your input and test it.


```python
naas.webhook.add()
```

### Execute webhook and get_output to know the format of parameters injected.


```python
naas.webhook.get_output()
```

### Thanks to the output notebook, you can understand the structure of your input (standard : params, headers, body).
Inject it in the blank cell in Step 1 to build your pipeline.


```python
EMAIL = body.get("Email")
EMAIL
```

### Build whatever you need. 
In this example, I want to send an email with the notebook template to the subscriber.

#### Setup a notification system.


```python
import naas
email_to = ["jeremy.ravenel@cashstory.com", EMAIL]
subject = "Get Naas Webhook tutorial template"
content = f"Hey {EMAIL},<br>Here is the Naas webhook tutorial template : https://public.naas.ai/amVyZW15LTJFbmFhczQyLTQwZ21haWwtMkVjb20=/asset/7c9359cbc967afd01d8e45b68659b3b0db4179582561f6fab70f156c460a"
naas.notifications.send(email_to=email_to, subject=subject, html=content)
```

## Output

### Share the notebook as an asset. 


```python
naas.assets.add()
```
