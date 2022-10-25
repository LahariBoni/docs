<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Slack/Slack_Send_message.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Slack+-+Send+message:+Error+short+description">🚨 Bug report</a>

**Tags:** #slack #message #send #operations #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook send message to a slack channel.

## Input

### Import needed library


```python
from naas_drivers import slack
```

### Setup Slack
- Create [Slack App](https://api.slack.com/apps)
- Add OAuth & Permissions(chat:write, chat:write.public to send message in a public channel)
- Install your App
- Get your Bot Token


```python
SLACK_TOKEN = "xoxb-XXXXXXXXXXXXXXXXXXXXXXX"
SLACK_CHANNEL = "channel-name"
SLACK_MESSAGE = "Hello World!"
```

## Model

### Connect to Slack


```python
SLACK = slack.connect(SLACK_TOKEN)
```

## Output

### Send your message 


```python
SLACK.send(SLACK_CHANNEL, SLACK_MESSAGE)
```
