<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Affinity/Affinity_Sync_with_Notion_database.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Affinity+-+Sync+with+Notion+database:+Error+short+description">🚨 Bug report</a>

**Tags:** #automation #notification #Affinity #Notion

**Author:** [Maxime Jublou](https://linkedin.com/in/maximejublou)

This notebook can be used to sync an Affinity list with a Notion database.

## Input


```python
import naas
from rich import print
import pandas as pd
from pandas import DataFrame as as_df
```

### Setup Notion
<a href='https://docs.naas.ai/drivers/notion'>How to get your Notion integration token ?</a>


```python
from naas_drivers import notion
from notion_client import APIResponseError

NOTION_TOKEN = naas.secret.get('tf_notion_token') or 'YOUR_NOTION_TOKEN'
NOTION_DATABASE_URL = 'https://www.notion.so/naas-official/16b30b9e11a14b138d1b0b37bc4a523e?v=9ebbb004b8484523828f619993cc424f'

notion.connect(NOTION_TOKEN)
```

### Setup Affinity

To get your token you simply need to go on Affinity website and then Settings > API. From there you will be able generate and get your API Token.


```python
import requests
import pydash
from requests.auth import HTTPBasicAuth

AFFINITY_API_KEY = naas.secret.get('AFFINITY_API_KEY') or 'YOUR_AFFINITY_API_KEY'
AFFINITY_LIST_NAME = "Naas"
```

### Setup Notifications


```python
from string import Template

PERSON_TO_NOTIFY = 'maxime@naas.ai'

NOTIFICATION_SUBJECT = Template("Notion page for $entry_name created ✅")

NOTIFICATION_MESSAGE = Template("""
<h1>👋 Hello, a new Notion page as been created for $entry_name.</h1>
</br>
<p>You can check it here: <a href="$notion_page_url">Notion page - $entry_name</a></p>
</br>
<p>
Best,</br>
Your naas automation 💚.
</p>
""")
```

### Setup Replication

You need to specify three things:
- Select a field to be a common identifier between Affinity and Notion.
- Which fields to sync from Affinity to Notion
- Which fields to sync from Notion to Affinity

#### Affinity

You need to specify `affinity` which can be:
- entry.entity.name
- entry.entity.domain
- Any column name with single values (no multi select)

#### Notion

You need to specify:
- `notion` which is the name of a property.


```python
IDENTIFIERS = {
    "affinity": 'entry.entity.domain',
    "notion": "Domain",
}

SYNC_ALL_AFFINITY_FIELDS = False
AFFINITY_FIELDS = ["Owners", "Status"]

SYNC_ALL_NOTION_FIELDS = False
NOTION_FIELDS = ["Status"]
```

## Model

### Get Notion database


```python
database = notion.database.get(NOTION_DATABASE_URL)
database.df()
```

### Create Affinity client


```python
# This is a future naas driver for Affinity.

class AffinityFieldValueType:
    PERSON: int = 0
    ORGANIZATION:int = 1
    DROPDOWN:int = 2
    NUMBER:int = 3
    DATE:int = 4
    LOCATION:int = 5
    TEXT:int = 6
    RANKED_DROPDOWN:int = 7
    OPPORTUNITY:int = 8

class Affinity():
    __api_key : str
    __api_url : str
    __basic_auth : HTTPBasicAuth
    __whoami : object
    __tenant_subdomain : str

    def __init__(self, api_url:str = "https://api.affinity.co", debug:bool = False):
        self.__api_url = api_url
        self.debug = debug
        
    def connect(self, api_key:str):
        self.__api_key = api_key
        self.__basic_auth = HTTPBasicAuth('', self.__api_key)
        
        self.__whoami = self.whoami()
        self.__tenant_subdomain = pydash.get(self.__whoami, 'tenant.subdomain')
        
    
    def __get(self, path):
        if self.debug:
            print(f'GET: {path}')
        response = requests.get(f'{self.__api_url}{path}', auth=self.__basic_auth)
        response.raise_for_status()
        return response.json()
    
    def __post(self, path, json_object):        
        if self.debug:
            print(f'POST: {path} {json_object}')
        response = requests.post(f'{self.__api_url}{path}', json=json_object, auth=self.__basic_auth)
        response.raise_for_status()
        return response.json()
    
    def __put(self, path, json_object):        
        if self.debug:
            print(f'PUT: {path} {json_object}')
        response = requests.put(f'{self.__api_url}{path}', json=json_object, auth=self.__basic_auth)
        response.raise_for_status()
        return response.json()
    
    def get_list_as_df(self, list_id:int):
        pass
    
    def whoami(self):
        return self.__get('/auth/whoami')
    
    def get_workspace_url(self):
        return f'https://{self.__tenant_subdomain}.affinity.co'
    
    def fields(self):
        return self.__get('/fields')
    
    def fields_by_id(self):
        fields = self.fields()
        return dict(zip(pydash.map_(fields, lambda x: x['id']), fields))
    
    def lists(self):
        return self.__get('/lists')
    
    def list_by_name(self, name:str):
        lists = self.__get('/lists')
        for l in lists:
            if l['name'] == name:
                return l
        return None
    
    def lists_entries(self, list_id:int):
        return self.__get(f'/lists/{list_id}/list-entries')
    
    def field_values(self, entry : object):
        entity_type = pydash.get(entry, 'entity_type')
        entity_id = pydash.get(entry, 'entity_id')
        
        if entity_type is None:
            raise Exception('entity_type not present in entry.')
        if entity_id is None:
            raise Exception('entity_id not present in entry.')
        
        parameter_name = None
        if entity_type not in [
            AffinityFieldValueType.PERSON,
            AffinityFieldValueType.ORGANIZATION,
            AffinityFieldValueType.OPPORTUNITY
        ]:
            parameter_name = 'list_entry_id'
        else:
            if entity_type == AffinityFieldValueType.ORGANIZATION:
                parameter_name = 'organization_id'
            elif entity_type == AffinityFieldValueType.PERSON:
                parameter_name = "person_id"
            elif entity_type == AffinityFieldValueType.OPPORTUNITY:
                parameter_name = "opportunity_id"
        
        
        return self.__get(f'/field-values?{parameter_name}={entity_id}')
    
    def field_values_by_field_id(self, entry : object):
        field_values = self.field_values(entry)
        
        values_by_fields = {}
        for fv in field_values:
            if fv['value'] is None:
                continue
            fv_field_id = fv['field_id']
            if fv_field_id not in values_by_fields:
                values_by_fields[fv_field_id] = []
                
            values_by_fields[fv_field_id].append(fv['value'])
        return values_by_fields
        

    def create_field(self, name:str, entity_type:int = None, value_type:int = None, list_id:int = None, allows_multiple:bool = None, is_list_specific:bool = None, is_required:bool = None):
        return self.__post('/fields', {
            'name': name,
            'entity_type': entity_type,
            'value_type': value_type,
            'list_id': list_id,
            'allows_multiple': allows_multiple,
            'is_list_specific': is_list_specific,
            'is_required': is_required
            
        })
    
    def create_new_field_value(self, field_id:int, entity_id:int, value:any, list_entry_id:int = None):
        return self.__post('/field-values', {
            'field_id': field_id,
            'entity_id': entity_id,
            'value': value,
            'list_entry_id': list_entry_id
        })

    def update_field_value(self, field_id:int, value:object) -> object:
        return self.__put(f'/field-values/{field_id}', {
            'value': value,
        })
    
    
    def person(self, person_id):
        return self.__get(f'/persons/{person_id}')
    
    def field_value_to_str(self, field, fv):
        value_type = field['value_type']
        if value_type == AffinityFieldValueType.RANKED_DROPDOWN:
            return fv['text']
        elif value_type == AffinityFieldValueType.PERSON:
            person = self.person(fv)
            #return f'{person["first_name"]} {person["last_name"]} {person["primary_email"]} {self.get_workspace_url()}/persons/{fv}'
            return f'{person["first_name"]} {person["last_name"]}'
        elif value_type in [
            AffinityFieldValueType.NUMBER,
            AffinityFieldValueType.TEXT,
            AffinityFieldValueType.DROPDOWN,
            AffinityFieldValueType.DATE,
            AffinityFieldValueType.ORGANIZATION, # Needs to be tested
            AffinityFieldValueType.OPPORTUNITY # Needs to be tested
        ]:
            return fv
        elif value_type == AffinityFieldValueType.LOCATION:
            values = []
            for i in fv:
                if fv[i] is None:
                    continue
                values.append(fv[i])
        
            return ", ".join(values)
    
    def str_to_field_value(self, value, fv):
        pass

affinityClient = Affinity(debug=True)
```

### Connect to affinity


```python
affinityClient.connect(AFFINITY_API_KEY)
```

### Get Affinity list and show entries


```python
affinityList = affinityClient.list_by_name(AFFINITY_LIST_NAME)

# Display entries
as_df(
    pydash.map_(
        affinityClient.lists_entries(affinityList['id']), lambda x: x['entity']
    )
)
```

### Affinity type to Notion type


```python
# Affinity type to Notion type rel map.
aftype_to_ntype = {
    AffinityFieldValueType.PERSON: 'select',
    AffinityFieldValueType.ORGANIZATION: 'number',
    AffinityFieldValueType.DROPDOWN: 'multi_select',
    AffinityFieldValueType.NUMBER: 'number',
    AffinityFieldValueType.DATE: 'date',
    AffinityFieldValueType.LOCATION: 'rich_text',
    AffinityFieldValueType.TEXT: 'rich_text',
    AffinityFieldValueType.RANKED_DROPDOWN: 'multi_select',
    AffinityFieldValueType.OPPORTUNITY: 'number'
}

def affinity_type_to_notion_type(affinity_type:int, allows_multiple:bool = True) -> str:
    if affinity_type not in aftype_to_ntype:
        raise Exception(f'Value type {value_type} not handled.')
    if affinity_type in [AffinityFieldValueType.DROPDOWN, AffinityFieldValueType.RANKED_DROPDOWN] and allows_multiple == False:
        return 'select'
    return aftype_to_ntype[affinity_type]
```

### Make sure "Notion Page" field exist in Affinity and create it otherwise


```python
notion_page_field_found = False
notion_page_field_id = None

# Get the list of fields
fields = affinityClient.fields()

for field in fields:
    if field['name'] == 'Notion Page':
        notion_page_field_found = True
        notion_page_field_id = field['id']
        break

if not notion_page_field_found:
    # Create the field
    new_field = affinityClient.create_field(
        name="Notion Page",
        entity_type=AffinityFieldValueType.ORGANIZATION,
        value_type=AffinityFieldValueType.TEXT,
        allows_multiple=False,
        is_list_specific=False,
        is_required=False
    )
    
    notion_page_field_id = new_field['id']
```

### Get fields by id


```python
fields_by_id = affinityClient.fields_by_id()
as_df(fields_by_id.values())
```

### Notification sender function


```python
def notify_notion_page_creation(entry_name:str, notion_page_url:str):
    
    subject = NOTIFICATION_SUBJECT.substitute(
        entry_name=entry_name
    )
    message = NOTIFICATION_MESSAGE.substitute(
        entry_name=entry_name,
        notion_page_url=notion_page_url
    )
    
    naas.notification.send(
    email_to=PERSON_TO_NOTIFY,
    subject=subject,
    html=message)
```

### Affinity get identifier value


```python
def get_affinity_identifier_value(affinityClient, identifier:str, entry:object, fields_by_id:object, values_by_fields:object):
    if identifier in ['entry.entity.name', 'entry.entity.domain']:
        return pydash.get({
            'entry': entry
        }, identifier)
    else:
        for f in fields_by_id:
            if f['name'] == identifier:
                field_id = f['id']
                value = values_by_fields[field_id]
                return affinityClient.field_value_to_str(f, value)
    raise Exception(f'Your Affinity Identifier "{identifier}" was not found.')
                
        
```

### Create/Update Notion page for each entry in the list


```python
entries = affinityClient.lists_entries(affinityList['id'])

for entry in entries:
    entity = entry["entity"]
    entity_id = entry["entity"]["id"]
    
    print(f'🔨 Handling {entity["name"]}')
    
    # Get values of each field.
    values_by_fields = affinityClient.field_values_by_field_id(entry)
    
    affinity_identifier_value = get_affinity_identifier_value(affinityClient, IDENTIFIERS['affinity'], entry, fields_by_id, values_by_fields)

    # Makes sure we have a notion page for this entry.
    page = None
    notion_page_gets_created_now = False
    try:
        pages = database.query({
            "filter": {
                "and": [
                    {
                        "property": IDENTIFIERS['notion'],
                        database.properties[IDENTIFIERS['notion']].type: {
                            "equals": affinity_identifier_value
                        }
                    }
                ]
            }
        })
        if len(pages) == 0:
            raise Exception('💡 Notion page does not exist.')
        page = pages[0]
    except Exception as e:
        print(e)
        page = notion.page.create(database_id=NOTION_DATABASE_URL, title=entity['name'])
        notion_page_gets_created_now = True
        print(f'✅ New notion page created for {entity["name"]}')
        print(f'💡 Sending email to notify page creation')
        notify_notion_page_creation(entity['name'], page.url) 
    
    # Make sure our identifier property exists in notion and have the proper value.
    if IDENTIFIERS['affinity'] == 'entry.entity.name':
        page.rich_text(IDENTIFIERS['notion'], affinity_identifier_value)
    elif IDENTIFIERS['affinity'] == 'entry.entity.domain':
        page.link(IDENTIFIERS['notion'], affinity_identifier_value)
    else:
        field = pydash.find(fields_by_id, lambda x: x['name'] == IDENTIFIERS['affinity'])
        field_name = field['name']

        notion_type = affinity_type_to_notion_type(field['value_type'], pydash.get(field, 'allows_multiple', False))

        getattr(page, notion_type)(field_name, affinity_identifier_value)
    
    # Sync Notion fields back to Affinity
    if SYNC_ALL_NOTION_FIELDS is True:
        pass
    elif notion_page_gets_created_now == False:
        for field_name in NOTION_FIELDS:
            # Skip the field if it's not present. Might happen on page creation.
            if field_name not in page.properties:
                continue
                
            # Gets notion property value
            notion_prop_value = str(page.properties[field_name])
    
            field = pydash.find(affinityClient.fields(), lambda x: x['name'] == field_name and x['list_id'] in [None, affinityList['id']])

            target_value = None
            if field['value_type'] in [AffinityFieldValueType.RANKED_DROPDOWN, AffinityFieldValueType.DROPDOWN]:
                # Find Dropdown value that matches our Notion property value.
                target_value = pydash.find(field['dropdown_options'], lambda x: x['text'] == notion_prop_value)
            else:
                target_value = notion_prop_value

            fv = pydash.find(affinityClient.field_values(entry), lambda x: x['field_id'] == field['id'])

            try:
                affinityClient.update_field_value(fv['id'], target_value['id'])
            except Exception as e:
                print(f'An error occured while sync')
                print(e)
    else:
        print('Skipping Notion sync as page just got created.')
            
            
    
    if SYNC_ALL_AFFINITY_FIELDS is True:
        # Create Notion page properties for each field value.
        for field_id in values_by_fields:
            values = values_by_fields[field_id]
            field = fields_by_id[field_id]
            field_name = field['name']

            # Get Notion property type corresponding to Affinity value type.
            notion_type = affinity_type_to_notion_type(field['value_type'], pydash.get(field, 'allows_multiple', False))

            # Convert Affinity values to be accepted by Notion's api.
            converted_values = [affinityClient.field_value_to_str(field, value) for value in values]

            fn = getattr(page, notion_type)
            if notion_type == "multi_select":
                fn(field_name, converted_values)
            else:
                fn(field_name, converted_values[0])
    else:
        for af in AFFINITY_FIELDS:
            field = pydash.find(fields_by_id, lambda x: x['name'] == af)
            field_id = field['id']
            values = values_by_fields[field_id]
            field_name = field['name']
            
            # Get Notion property type corresponding to Affinity value type.
            notion_type = affinity_type_to_notion_type(field['value_type'], pydash.get(field, 'allows_multiple', False))
            
            # Convert Affinity values to be accepted by Notion's api.
            converted_values = [affinityClient.field_value_to_str(field, value) for value in values]
            
            fn = getattr(page, notion_type)
            if notion_type == "multi_select":
                fn(field_name, converted_values)
            else:
                fn(field_name, converted_values[0])
    

    # Update Notion page.
    page.update()
    print('✅ Notion page updated')

    # If "Notion Page" not referenced in Affinity we add it for this entry.
    if notion_page_field_id not in values_by_fields:
        res = affinityClient.create_new_field_value(
        field_id=notion_page_field_id, 
        entity_id=entity_id,
        value=page.url
        )
        print('✅ Notion page linked into Affinity')
    elif values_by_fields[notion_page_field_id][0] != page.url:
        # Fix Notion link in Affinity if it does not match the actual page.
        all_field_values = affinityClient.field_values(entry)
        for fv in all_field_values:
            if fv['field_id'] == notion_page_field_id:
                res = affinityClient.update_field_value(
                field_id=fv['id'], 
                value=page.url
                )
                print('✅ Notion page linked into Affinity')
    print(f'{entity["name"]} handling completed 👍\n\n')
        
```

## Output


```python
database.df()
```

## Schedule Notebook to run every 15 minutes


```python
naas.scheduler.add(cron="*/15 * * * *")
```


```python
#naas.scheduler.delete()
```
