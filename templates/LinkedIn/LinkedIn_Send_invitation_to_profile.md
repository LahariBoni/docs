<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Send_invitation_to_profile.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Send+invitation+to+profile:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #invitation #naas_drivers #content #snippet #text

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

Growth your network and send invitation to profile.

## Input

### Import library


```python
from naas_drivers import linkedin
import naas
```

### Setup LinkedIn

If you are using the Chrome Extension:

- [Install Naas Chrome Extension](https://chrome.google.com/webstore/detail/naas/cpkgfedlkfiknjpkmhcglmjiefnechpp?hl=fr&authuser=0)
- [Create a new token](https://app.naas.ai/hub/token)
- Copy/Paste your token in your extension
- Login/Logout your LinkedIn account
- Your secrets "LINKEDIN_LI_AT" and "LINKEDIN_JSESSIONID" will be added directly on your naas everytime you login and logout.

or <br>

If you are not using the Google Chrome Extension, [learn how to get your cookies on LinkedIn](https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75) and set up the values below:
- 🍪 li_at
- 🍪 JSESSIONID


```python
# Cookies
LI_AT = naas.secret.get("LINKEDIN_LI_AT") or "li_at"
JSESSIONID = naas.secret.get("LINKEDIN_JSESSIONID") or "JSESSIONID"

# Profile URL you want to send the invitation to
recipient_url = "https://www.linkedin.com/in/****/"

# Message to add with your invitation
message = "Hello, \nI will be happy to connect!"
```

## Model

### Send invitation


```python
result = linkedin.invitation.send(recipient_url=RECIPIENT_URL, message=message)
```

## Output

### Display result


```python
result
```
