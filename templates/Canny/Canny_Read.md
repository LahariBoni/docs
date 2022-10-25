<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Canny/Canny_Read.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Canny+-+Read:+Error+short+description">🚨 Bug report</a>

**Tags:** #canny #product #operations #snippet

**Author:** [Martin Donadieu](https://www.linkedin.com/in/martindonadieu)

## Enter credentials

## Input

### Import librairies


```python
import requests
import json
import pandas as pd
```

### API key


```python
canny_api = "CANNY_API_KEY"   # api key of canny
```

## Model

### Connecting to canny


```python
class canny:
    def __init__(self,api_key):
        self.api_key = api_key

    def read(self):
        canny_api = self.api_key
        response = requests.get("https://canny.io/api/v1/posts/list")
        api_key = {
        "apiKey":canny_api          
        }
        board_id = {
        "id":""                          
        }
        limit = {
        "limit":"100"                          
        }
        data = {**api_key, **board_id, **limit}
        response = requests.post("https://canny.io/api/v1/posts/list",data)
        post_details = response.json()
        pd.set_option('mode.chained_assignment', None)
        dd = post_details['posts']
        df = pd.DataFrame(columns = dd[0].keys()) 
        for i in range(len(dd)):
            df = df.append(dd[i], ignore_index=True)
        df = df.rename(columns={'details': 'POST_DETAIL', 'status': 'STATUS', 'title': 'POST_NAME','board': 'BOARD','category': 'CATEGORY','id': 'BOARD_ID'})        
        board = []
        category = []
        tags = []
        eta = []
        created = []
        for i in range(len(df)):
            board.append(df['BOARD'][i]['name'])
            created.append(df['BOARD'][i]['created'])
            if not df['CATEGORY'][i]:
                category.append('Not assigned')
            else:
                category.append(df['CATEGORY'][i]['name'])    
            if not df['tags'][i]:
                tags.append('Not assigned')
            else:
                tags.append(df['tags'][i][0]['name'])
            if not df['eta'][i]:
                eta.append('Not assigned')
            else:
                eta.append(df['eta'][i])  
        df1 = df[['POST_NAME','POST_DETAIL','STATUS','BOARD_ID']]
        df1['BOARD'] = board
        df1['CREATED'] = created
        df1['ETA'] = eta
        df1['CATEGORY'] = category
        df1['TAGS'] = tags
        return df1
```

### Post Retrieve


```python
canny(canny_api).read()
```

## Output

### Save as csv


```python
canny(canny_api).read().to_csv('naas_canny.csv') 
```
