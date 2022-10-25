<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Send_Slack_Messages_For_New_Notion_Database_Items.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Send+Slack+Messages+For+New++Database+Items:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #slack #operations #automation


**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)

This notebook sends a Slack message every-time it sees a new notion page added to a database.
<br/>References :
- Notion templates : [https://docs.naas.ai/templates/notion](https://docs.naas.ai/templates/notion)
- Create new slack app : [https://api.slack.com/authentication/basics](https://api.slack.com/authentication/basics)

- For this use case we need to create & use user token, rather than bot token with the following permissions/scopes -> [channels: history, channels: read, chat: write, users: read]

## Input


### Import libraries



```python
from naas_drivers import notion, slack
import naas
```

### Setup Notion

- [Get your Notion integration token](https://docs.naas.ai/drivers/notion)
- Share integration with your database



```python
# Enter Token API
NOTION_TOKEN = "secret_J9JIQksrylGmJpErmw49A7U9ON1lIdGLjbVk6tDFh2y"

# Enter Database id
DATABASE_ID = "https://www.notion.so/naas-official/72c87516d6e1419fb3a69763892898c7?v=2e71afc61e7644409dd874957c98e78e"
```

### Setup Slack



```python
# Token
SLACK_TOKEN = "xoxb-xxx-xxx-xxx"

# Channel name
SLACK_CHANNEL = "channel-name"
```

### Setup Naas



```python
# Schedule your notebook every 15min
naas.scheduler.add(cron="*/15 * * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()

```

## Model


### Get pages from Notion database



```python
pages = notion.connect(NOTION_TOKEN).database.query(DATABASE_ID, query={})
```

## Output


### Send messages to Slack



```python
def send_message():

    for page in pages:
        # For the first time ever if there is no property/column named 'slack notification sent' in database 
        if "Slack notification sent" not in page.properties.keys():
            page_name = page.properties["Name"]
            page_url = page.url

            slack.connect(SLACK_TOKEN).send(SLACK_CHANNEL, f'New notion page created "{page_name}" here: {page_url}')
            page.select('Slack notification sent', "True")
            page.update()

            print(f'✅ Notification sent for {page_name}: {page_url}')
        
        # If there is a column present then checks if it is False or None and updates
        else:
            if str(page.properties['Slack notification sent'])!='True':
                page_name = page.properties["Name"]
                page_url = page.url

                slack.connect(SLACK_TOKEN).send(SLACK_CHANNEL, f'New notion page created "{page_name}" here: {page_url}')
                page.select('Slack notification sent', "True")
                page.update()
                print(f'✅ Notification sent for {page_name}: {page_url}')
                
send_message()
```
