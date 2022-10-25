<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Typeform/Typeform_Log_New_Typeform_Entries_In_Notion_Databases.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Typeform+-+Log+New++Entries+In+Notion+Databases:+Error+short+description">🚨 Bug report</a>

**Tags:** #typeform #notion #operations #automation


**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190)

Capture and organize important project and customer details without slowing down for tedious copy-and-paste tasks! This template automatically adds each new entry received in Typeform as an item in your Notion database.
<br/>References :
- [TypeForm API](https://developer.typeform.com/responses/)
- [Notion API ](https://docs.naas.ai/drivers/notion)

Have a look at the sample typeform format used [here](https://1mx5hrd76qo.typeform.com/to/bw1oM4SP)

Sample output view of the notion database [here](https://billowy-lemming-95e.notion.site/f8e44ff261564c76b3bb80e6edb171a9?v=1d2a506563fe4082b71e78695185962e), which has all the questions asked in the typeform as column names and their responses as entries.

This output database consists of only 5 responses collected over the sample typeform.

## Input


### Import libraries



```python
from naas_drivers import notion
from typeform import Typeform
import naas, pandas as pd
import requests
from datetime import datetime
import pydash as pd_, re
```

### Setup Typeform

- Get your [access token](https://developer.typeform.com/get-started/personal-access-token)



```python
TYPEFORM_ACCESS_TOKEN = "<TYPEFORM_ACCESS_TOKEN>"

# Unique ID for the form. Find in your form URL.
# For example, in the URL "https://mysite.typeform.com/to/u6nXL7" the
# form_id is u6nXL7.

FORM_ID = "<FORM_ID>"

typeform = Typeform(TYPEFORM_ACCESS_TOKEN)
```

### Setup Notion

- Get your [Notion integration token ?](https://docs.naas.ai/drivers/notion)
- Share you database with your integration



```python
NOTION_TOKEN = "<NOTION_TOKEN>"
# database-url
# https://www.notion.so/naas-official/f89fddc31128400fab11001a215aff09?v=d84b89b704c7dssd432350cc273
DATABASE_URL = "<DATABASE_URL>"
```

### Setup Naas



```python
# Schedule your notebook everyday at 9 AM
naas.scheduler.add(cron="0 9 * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()

```

## Model


### Get data from Typeform

- Currently supported data types for retreiving data from typeform with this notebook are as follows:
  - Text (short and long)
  - Number
  - Rating
  - Multi-choice with one or more correct responses
  - Phone number
  - Email

**Collecting questions from typeform to setup as column names in database**


```python
def get_questions_dict():
    fields = typeform.forms.get(FORM_ID)['fields']
    questions={}
    for field in fields:
        new_str = re.sub('{{field:' + r'[0-9A-Z]+'+'}}, ', '', field['title'])
        questions[field['id']] = new_str
    return questions

questions = get_questions_dict()
questions
```

**Collecting answers from typeform**


```python
def get_answers(form_id, token):
    url = f"https://api.typeform.com/forms/{form_id}/responses"
    headers = {'Authorization': f'Bearer {token}'}

    r = requests.get(url, headers = headers)
    responses = pd_.get(r.json(), 'items')
    answers= []
    for resp in responses:
        response_answer =[]
        for field in pd_.get(resp, 'answers'):
            entries={}
            entries['id'], entries['type'] = pd_.get(field, 'field.id'), pd_.get(field, 'type')
    
            if entries['type'] == 'text':
                entries['answer'] = pd_.get(field, f"{entries['type']}")
                
            elif entries['type'] == 'choice':
                entries['answer'] = pd_.get(field, 'choice.label')
                
            elif entries['type'] == 'choices':
                entries['answer'] = pd_.get(field, 'choices.labels')
                
            elif entries['type'] == 'number':
                entries['answer'] = pd_.get(field, 'number')
                
            elif entries['type'] == 'phone_number':
                entries['answer'] = pd_.get(field, 'phone_number')
            
            elif entries['type'] == 'email':
                entries['answer'] = pd_.get(field, 'email')
                
            response_answer.append(entries)
        answers.append(response_answer)
    return answers

answers = get_answers(FORM_ID, TYPEFORM_ACCESS_TOKEN)
answers[0]
```


```python
def get_typeform_data(form_id, token, questions, answers):
    # Get response dataframe
    # Retrieve response and map field id with field title to column name's
    df = pd.DataFrame()
    url = f"https://api.typeform.com/forms/{form_id}/responses"
    headers = {'Authorization': f'Bearer {token}'}

    r = requests.get(url, headers = headers)
    lst_responses = pd_.get(r.json(), 'items')
    for idx, resp in enumerate(lst_responses):
        in_time, out_time = pd_.get(resp, 'landed_at'), pd_.get(resp, 'submitted_at')
        time_diff = datetime.strptime(out_time, '%Y-%m-%dT%H:%M:%SZ') - datetime.strptime(in_time, '%Y-%m-%dT%H:%M:%SZ')
        if str(time_diff).split(':')[1] == '00':
            df.loc[idx, 'time_taken_to_fill_form'] = ":".join(str(time_diff).split(':')[1:]) + 'secs'
        else:
            df.loc[idx, 'time_taken_to_fill_form'] = ":".join(str(time_diff).split(':')[1:]) + 'mins'
        
        df.loc[idx, 'response_id'] = pd_.get(resp, 'response_id')
    
    for idx, response in enumerate(answers):
        for entity in response:
            if entity['type'] == 'choices':
                df.loc[idx, questions[entity['id']]] = ",".join(entity['answer'])
            else:
                df.loc[idx, questions[entity['id']]] = entity['answer']
                
    df.fillna('None', inplace=True)
    
    return df

df_typeform = get_typeform_data(FORM_ID, TYPEFORM_ACCESS_TOKEN, questions, answers)
df_typeform
```

### Get data from Notion DB



```python
pages = notion.connect(NOTION_TOKEN).database.query(DATABASE_URL, query={})
len(pages)
```

### Adding data to Notion



```python
def add_new_entries(df):
    if df.shape[0]==0:
        return df
    
    columns = df.columns.to_list()
    for col in columns:
        if 'name' in col:
            name_col = col
    
    columns.remove(name_col)
    
    for idx, row in df.iterrows():
        if idx == df.shape[0]:
            break
        page = notion.connect(NOTION_TOKEN).page.create(database_id=DATABASE_URL, title= row[name_col])
        for column in columns:
            page.rich_text(column, str(row[column]))
            page.update()
    return df
```


```python
def add_data_to_notion(df_typeform, pages):
    id_present = False
    try:
        pages[0].properties['response_id']
        id_present = True
    except KeyError:
        pass
    
    # If no data is present initially
    if not id_present:
        df = add_new_entries(df_typeform)
    
    # If some data exists
    else:
        notion_df = notion.connect(NOTION_TOKEN).database.get(DATABASE_URL).df()
        existing_ids = notion_df.response_id.to_list()
        
        new_entries = df_typeform[df_typeform.response_id.isin(existing_ids) == False]
        df = add_new_entries(new_entries)
    
    return df

df_notion = add_data_to_notion(df_typeform, pages)
```

## Output



```python
df_notion.head()
```
