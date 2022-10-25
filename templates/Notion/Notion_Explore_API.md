<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Explore_API.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Explore+API:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #productivity #operations #snippet

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

This notebook is an exploration of what you can do with the Notion's API.

Resources: 
- Notion official documentation : https://developers.notion.com/
- Youtube video (not official) : https://www.youtube.com/watch?v=sdn1HgxLwEg

## Input


```python
import requests
import pandas as pd
import json
```

### Setting up Notion connections

1. Create a page or a database on Notion.
2. Create an integration in your workspace.
3. Share your page or database to this integration.

More explanation on how to do this here: https://developers.notion.com/docs/getting-started


```python
DATABASE_ID_TEST = "a296bd16b7284bc494aa91f50ad64d30" #https://www.notion.so/a296bd16b7284bc494aa91f50ad64d30?v=d37af84a3a6744fb957002073a267c44

PAGE_ID = "e2e8b31737174dbe86b9ae65f9b8eb9c" #click on Page and Get ID : https://www.notion.so/Mary-Meeks-2d822179eb59451e91e83086cdd74e5c

INTEGRATION_TOKEN = "secret_gF6bJPSyOgt5oZgb2sgT1yiMxfS4LqNmWmd2M8S5vzl"
```


```python
NOTION_DB_URL = "https://api.notion.com/v1/databases/"

NOTION_PAGE_URL = "https://api.notion.com/v1/pages/"

NOTION_PAGE_CONTENT = "https://api.notion.com/v1/blocks/"
```

## Model

### Get database properties

More information here: https://developers.notion.com/reference/get-database


```python
database_url = NOTION_DB_URL + DATABASE_ID_TEST 

response = requests.get(database_url, headers={"Authorization": f"{INTEGRATION_TOKEN}"})
print (response.json())
```

More information here: https://developers.notion.com/reference/post-database-query


```python
database_url = NOTION_DB_URL + DATABASE_ID_TEST + "/query"

query = {"filter": {"property": "High Priority", "checkbox": {"equals": True}}}
query = {"filter": {"property": "Cost of next trip", "number": {"greater_than_or_equal_to": 0.5}}}

headers = {"Authorization": f"{INTEGRATION_TOKEN}", "Notion-Version": "2021-05-13"}

response = requests.post(database_url, headers=headers, data=query)
print((response.json()['results']))
```


```python
df_structure = pd.DataFrame.from_dict(response.json()['results'])
```


```python
print("The size of the df is", df_structure.shape)
df_structure.head()
```

⚠️ Notion's API allows us to retrieve a maximum of 100 records. So if your base is bigger than 100 records it will only retrieve the 100 last edited ones. 

⚠️ If your database has some relation to some other databases, think to share the linked databases with the integration aswell.

As we can see the content of the Notion table is in the properties column. We will now extract it and see what it contains.

The column properties contain a dictionary for each Notion record. We will exctract each of these disctionnaries and create a new dataFrame. 


```python
list_dict = []
for index, row in df_structure.iterrows():
    list_dict.append(row['properties'])

temp_df = pd.DataFrame.from_dict(list_dict)
```


```python
# Get the columns name in a list to use them later
columns = temp_df.columns.values.tolist()
```


```python
temp_df.head()
```

As we can see, each of the properties contains another dict of the information. 

Let's see how the dictionnary containing the data is structured.


```python
for index, value in temp_df.iloc[2].items():
    print(value)
```


```python
pd.DataFrame.from_dict(list_dict).iloc[0]['Name']
```

Let's create a small function to extract the data. 

All the properties contain an id and a type. The type will then be used to find the original information of the property.

Sometimes, the data will be contained directly as a string, sometimes it will be a dict sometimes it will be a list of dict. 

### Query database


```python
def extract_name_or_plaintext(dictionnary):
    # Given a dictionnary it will output the string of the key name or plain_text
    if 'name' in dictionnary.keys():
        return dictionnary['name']
    elif 'plain_text' in dictionnary.keys():
        return dictionnary['plain_text']
    else:
        return ''

def extract_date(dictionnary):
    # For the moment we extract only the starting date of a date field
    # Example {'id': 'prop_1', 'type': 'date', 'date': {'start': '2018-03-21', 'end': None}}
    # Input : {'start': '2018-03-21', 'end': None}
    return dictionnary['start']
```


```python
def extract_data(element):
    # input: a dictionnary of a notion property
    # Exemple: {'id': 'W#4k', 'type': 'select', 'select': {'id': 'b305bd26-****-****-****-c78e2034db8f', 'name': 'Client', 'color': 'green'}}
    # output: the string containing the information of the dict. (Client in the exemple)
    if type(element) is dict:
        dict_type = element['type'] 
        informations = element[dict_type]
        
        if dict_type == 'date':
            informations = extract_date(informations)
        
        elif type(informations) is dict:
            informations = extract_name_or_plaintext(informations)
        
        elif type(informations) is list:
            informations_temp = ''
            for element_of_informations_list in informations:
                informations_temp += extract_name_or_plaintext(element_of_informations_list) + ", "
            informations = informations_temp[:-2]
        return informations
    
    else:
        return ''
```


```python
all_list = []
for i in range (temp_df.shape[0]):
    temp_list = []
    for index, value in temp_df.iloc[i].items():
        temp_list.append(extract_data(value))
    all_list.append(temp_list)
df_content = pd.DataFrame.from_records(all_list, columns = columns)

```

### Get only visible headers


```python
df_content.head()
```

### Get full headers


```python
df_full = pd.concat([df_structure, df_content], axis=1)
```


```python
df_full
```

### Get page properties

There is two different API calls to interact with a page. 

* **Get a page** will give us a page properties: https://developers.notion.com/reference/get-page

* **Get block children** will give us the page content: https://developers.notion.com/reference/get-block-children



```python
page_url = NOTION_PAGE_URL + PAGE_ID 

response = requests.get(page_url, headers={"Authorization": f"{INTEGRATION_TOKEN}", "Notion-Version": "2021-05-13"})
print (response.json())
```

### Retrieve a page content


```python
page_url = NOTION_PAGE_CONTENT + PAGE_ID + "/children"
headers = {"Authorization": f"{INTEGRATION_TOKEN}", "Notion-Version": "2021-05-13"}

response = requests.get(page_url, headers=headers)
print(response.json())
```

Some types are accessible. But some other types are unsupported and cannot be read through the API. 

Some unsupported types : 
* image
* bookmarked link
* other page (it has an unsupported type but it will be readable through its page id)

### Create a record 


```python
page_url = NOTION_PAGE_URL
page_id = DATABASE_ID_TEST

# Surprisingly in this case you need to add "Content-Type": "application/json"
# If not you will an error 400: body failed validation: body.parent should be defined, instead was 'undefined'.
headers = {"Authorization": f"{INTEGRATION_TOKEN}", "Notion-Version": "2021-05-13", "Content-Type": "application/json"}

name = {"title":[{"text":{"content":"Added via API NAAS"}}]}
company = {"rich_text": [{"text": {"content": "Test Company"}}]}
status = {"select": {"name": "Lost"}}
est_value = {"number": 10000 }

header = {"object": "block",
          "type": "heading_2",
          "heading_2": {
            "text": [{ "type": "text", "text": { "content": "Naas API test" } }]
                }
             }

paragraph = {"object": "block",
             "type": "paragraph",
             "paragraph": {
               "text": [
                 {
                    "type": "text",
                    "text": {
                      "content": "Notebooks as a service for data geeks. Naas notebooks enable you to script faster with low-code formulas & templates. Automate all your tasks in minutes..",
                                }
                            }
                      ]
                    }
                }

to_do = {"object": "block",
             "type": "to_do",
             "to_do": {
               "text": [
                 {
                    "type": "text",
                    "text": {
                      "content": "Automate all your tasks in minutes..",
                                }
                            },
                 {
                    "type": "text",
                    "text": {
                      "content": "Script faster",
                                }
                            }
                      ]
                    }
                }
```

## Output

### Setup object to post


```python
myobj = {
          "parent": {"database_id": page_id}, 
          "properties":
            {
              "Name":name,
              "Company": company,
              "Status": status,
              "Estimated Value": est_value
            },
          "children":[header, paragraph,to_do]
        } 
```

### Post record


```python
data = json.dumps(myobj)

response = requests.post(page_url, headers=headers, data=data)

if 'status' in response.json().keys():
    if response.json()['status'] != 200:
        print ("Error:", response.json()['message'])
elif 'object' in response.json().keys(): 
    print("✅ Your data was added to Notion")
    print(response.json())
```
