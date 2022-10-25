<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Twilio/Twilio_Make_Call.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Twilio+-+Make+Call:+Error+short+description">🚨 Bug report</a>

**Tags:** #twilio #project #call #mobile

**Author:** [Sriniketh Jayasendil](https://www.linkedin.com/in/sriniketh-jayasendil/)

**Description:** This notebook allows us to make a phone call to a verified twilio number. It also has different parameters to customize the output of the voice call.

## Input

## Import libraries


```python
import time
import naas

try:
    from twilio.rest import Client
except ModuleNotFoundError:
    ! pip install twilio
    from twilio.rest import Client
```

### Setup Twilio


```python
account_sid = naas.secret.get(name="account_sid")    # From Twilio Console
auth_token = naas.secret.get(name="auth_token")     # From Twilio Console
from_number = naas.secret.get(name="from_number")    # From Twilio Console
to_number = naas.secret.get(name="to_number")      # Could send messages only to verified Caller ID's
```

## Model


### Connect to Twilio


```python
client = Client(account_sid, auth_token)  # Set Client
```

## Output


### Make call


```python
call = client.calls.create(
                        url='http://demo.twilio.com/docs/voice.xml',
                        to=to_number,
                        from_=from_number
                    )

time.sleep(5)
print(call.sid)
```
