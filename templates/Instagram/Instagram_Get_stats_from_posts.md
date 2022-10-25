<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Instagram/Instagram_Get_stats_from_posts.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Instagram+-+Get+stats+from+posts:+Error+short+description">🚨 Bug report</a>

**Tags:** #instagram #snippet #dataframe #content

**Author:** [Mohamed Abidi](https://www.linkedin.com/in/mohamed-abidi-919505192/)

## Input

### Import libraries


```python
import requests
import json
import datetime
import pandas as pd
```

### Setup Instagram 
Once you’ve made an Instagram Business Account, you need to make developer account in [Meta for developers](https://developers.facebook.com/)
After that, you need to make an app first to access insight data from Instagram. Your app type will determine which APIs are available to you.

Here below you will find every parameters you need to have before running your notebook : 

    - Access_token: short-lived access token that we got directly before we start writing our code
    - Client_id: App ID from Settings
    - Client_secret: App Secret from Settings
    - Page_id: Facebook Page ID from your business page’s ‘About’ tab
    - Instagram_account_id: shown during confirmation to generate token process
    
Permissions to ask for when generating a token:
- pages_show_list
- instagram_basic
- instagram_content_publish
- pages_read_engagement
- public_profile


```python
access_token = ""
client_id = ""
client_secret = ""
page_id = ""
instagram_account_id = ""
ig_username = "naaslife"
```

### Variables


```python
params = dict()
params['access_token'] = access_token        
params['client_id'] = client_id
params['client_secret'] = client_secret
params['graph_domain'] = 'https://graph.facebook.com'
params['graph_version'] = 'v13.0'
params['endpoint_base'] = params['graph_domain'] + '/' + params['graph_version'] + '/'
params['page_id'] = page_id           
params['instagram_account_id'] = instagram_account_id
params['ig_username'] = ig_username
```

### Check the expiration date of your token


```python
# Define Endpoint Parameters
endpointParams = dict()
endpointParams['input_token'] = params['access_token']
endpointParams['access_token'] = params['access_token']

# Define URL
url = params['graph_domain'] + '/debug_token'
# Requests Data
try:
    data = requests.get(url, endpointParams)
    access_token_data = json.loads(data.content)
    
except Exception as err:
    print(err)

print("Token Expires: ", datetime.datetime.fromtimestamp(
    access_token_data['data']['expires_at']))
```

## Model

### Get list of posts and associated stats


```python
# Define URL
url = params['endpoint_base'] + params['instagram_account_id'] + '/media'

# Define Endpoint Parameters
endpointParams = dict()
endpointParams['fields'] = 'id,caption,media_type,media_url,permalink,thumbnail_url,timestamp,username,like_count,comments_count'
endpointParams['access_token'] = params['access_token']

# Requests Data
data = requests.get(url, endpointParams)
basic_insight = json.loads(data.content)
```

## Output

### Display result


```python
df = pd.DataFrame(basic_insight['data'])
df.columns = ['id', 'Caption', 'Media_Type', 'Media_URL',
              'Permalink', 'Timestamp', 'Username', 'Likes', 'Comments']
df.head()
```
