<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Canny/Canny_Create.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Canny+-+Create:+Error+short+description">🚨 Bug report</a>

**Tags:** #canny #product #operations #snippet

**Author:** [Martin Donadieu](https://www.linkedin.com/in/martindonadieu)

## Input

### Import librairies


```python
import requests
import json
import pandas as pd
```

### Enter credentials


```python
canny_api = "CANNY_API_KEY"                          # api key of canny
post_title = "Post title"                            # Enter post title                    
post_body = "Post body using canny api"              # Enter post body
```

## Model

### Board dataframe using api-key


```python
api_key = {
"apiKey":canny_api          
}
limit = {
"limit":"100"                          
}
response = requests.get("https://canny.io/api/v1/boards/list")
response = requests.post("https://canny.io/api/v1/boards/list", api_key)
post_details = response.json()
db = post_details['boards']
df = pd.DataFrame(columns = db[0].keys()) 
for i in range(len(db)):
    df = df.append(db[i], ignore_index=True)
df = df[['name','id']]
board_list = df.rename(columns={'name': 'BOARD_NAME', 'id': 'BOARD_ID'})
board_list
```

### Enter board name


```python
board_name = "Requests"      #Enter board name
for i in range(len(board_list)):
    if board_list['BOARD_NAME'][i] == board_name:
        board_id = board_list['BOARD_ID'][i]
board_id
board_id = {
"boardID":board_id                          
}
```

### Using api and board name to get author list


```python
response = requests.get("https://canny.io/api/v1/posts/list")
data = {**api_key, **board_id, **limit}
response = requests.post("https://canny.io/api/v1/posts/list", data)
post_details = response.json()
# post_details['posts']
author_list = pd.DataFrame()
for i in range(len(post_details['posts'])):
    author_list = author_list.append(post_details['posts'][i]['author'], ignore_index=True)
author_list.drop_duplicates(subset ="email", keep = False, inplace = True)
author_list = author_list[['name','id']]
author_list = author_list.rename(columns={'name': 'AUTHOR_NAME', 'id': 'AUTHOR_ID'})
author_list
```

### Enter author name


```python
author_name = "Sanjay Sabu"  #Enter author name
for i in author_list['AUTHOR_NAME'].index:
    if author_list['AUTHOR_NAME'][i] == author_name:
        author_id = author_list['AUTHOR_ID'][i]
author_id = {
"authorID":author_id                          
}
```

### Creating post


```python
post_title = {
"title":post_title                                    
}
post_body = {
"details":post_body                    
}
data = {**api_key, **author_id, **board_id, **post_body, **post_title}
```

## Output

### Send the post


```python
response = requests.post("https://canny.io/api/v1/posts/create", data)
```
