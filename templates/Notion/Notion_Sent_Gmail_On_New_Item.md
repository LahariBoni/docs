<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Sent_Gmail_On_New_Item.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Sent+Gmail+On+New+Item:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #gsheet #productivity #naas_drivers #operations #automation #email

**Author:** [Arun K C](https://www.linkedin.com/in/arun-kc/)

## Input

### Import librairies

Let's import all the necessary libraries required


```python
import naas
from naas_drivers import notion, gsheet
from naas_drivers import html
import pandas as pd
```

### Variables

Replace all the variables below with appropriate values.


```python
# Notion
token = "NOTION_TOKEN"
database_id = "NOTION_DATABASE_ID"

# Gsheet
spreadsheet_id = "SPREADSHEET_ID"
mail_list_sheet_name = "Sheet1"
item_list_sheet_name = "Sheet2"
your_email = "YOUR_EMAIL_ID"
```

### Setting up the scheduler

Let's schedule the notebook for every 15mins ⏰

Ps: to remove the "Scheduler", just replace .add by .delete


```python
#Schedule the notebook to run every 15 minutes
naas.scheduler.add(cron="*/15 * * * *")
```

## Model

### Read the gsheet


```python
email_list_data = gsheet.connect(spreadsheet_id).get(sheet_name = mail_list_sheet_name)
try:
    item_list_history = gsheet.connect(spreadsheet_id).get(sheet_name = item_list_sheet_name)
except:
    item_list_history = []
```

### Setting up email


```python
firstname_list = email_list_data['FIRSTNAME']
email_list = email_list_data['EMAIL']
```

### Get database from notion


```python
def create_notion_connection():
    database = notion.connect(token).database.get(database_id)
    df_db = database.df()
    print(df_db)
    return df_db
```

### Send data to Gsheet


```python
#Send data to Gsheet
def send_data_to_gsheet(data):
    gsheet.connect(spreadsheet_id)
    gsheet.send(
        sheet_name = item_list_sheet_name,
        data = data
    )
```

### Get new items from Notion

Let's fetch out the new items from Notion

Here our unique key is **Id**


```python
#Get new notion items list 
def get_new_items_list(df_db):
    
    if not list(item_list_history):
        new_items = df_db
    else:
        item_list_history['Id'] = item_list_history['Id'].astype(int)
        df_db['Id'] = df_db['Id'].astype(int)

        common = df_db.merge(item_list_history, on=["Id"])
        new_items = df_db[~df_db.Id.isin(common.Id)]  
        
    data = [] 
    
    for i in range(len(new_items.index)):
        dictionary = {}
        for col in new_items.columns:
            dictionary[col] = str(new_items.iloc[i][col])          
        data.append(dictionary)
    
    send_data_to_gsheet(data)
    
    return data
    
```

### Create email content


```python
#Get email contents
def get_mail_content():
    email_content = html.generate( 
            display = 'iframe',
            title = 'Updates here!!',
            heading = 'Hi {first_name}, you have some new items in you notion list',
            text_1 = 'Following are the new list of items seperated by comma : ',
            text_2 = '{new_items_list}',
            text_3 = 'Have a great day!!'          
    )
    #print(email_content)
    return email_content
```

### Sending Emails


```python
#Send mail to recipients
def send_mail(new_items_list):
    email_content = get_mail_content()
    for i in range(len(email_list_data)):
        subject = "Update on Notion items"
        content = email_content.replace("{first_name}",firstname_list[i]).replace("{new_items_list}",new_items_list)
        naas.notifications.send(email_to=email_list[i], subject=subject, html=content, email_from=your_email)
```

## Output


```python
df = create_notion_connection()
```


```python
new_items_list = get_new_items_list(df)
new_items_list = ', '.join([data['Books'] for data in new_items_list])
```


```python
if new_items_list:
    send_mail(new_items_list)
else:
    print('No new items!!')
```
