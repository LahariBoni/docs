<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Plaid/Plaid_Get_accounts.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Plaid+-+Get+accounts:+Error+short+description">🚨 Bug report</a>

**Tags:** #plaid #bank #accounts #snippet #finance #dataframe

**Author:** [Martin Donadieu](https://www.linkedin.com/in/martindonadieu/)

## Input


```python
pip install plaid-python
```

### Create account here :
https://plaid.com/
    
    

### Import libraries


```python
import os
import plaid
import IPython.core.display
import uuid
import naas
import json
import pandas as pd
```

### Config your variables


```python
PLAID_CLIENT_ID = "*************" 
PLAID_SECRET = "*************" 
PLAID_ENV = 'sandbox'

PLAID_PRODUCTS = ['transactions']
PLAID_COUNTRY_CODES = ['FR']
start_transaction = "2020-09-01"
end_transaction = "2020-10-01"
```

## Model

### Connect to plaid


```python
client = plaid.Client(client_id=PLAID_CLIENT_ID,
                      secret=PLAID_SECRET,
                      environment=PLAID_ENV)
```


```python
def create_link_token():
    response = client.LinkToken.create(
      {
        'user': {
          # This should correspond to a unique id for the current user.
          'client_user_id': 'user-id',
        },
        'client_name': "Plaid Quickstart",
        'products': PLAID_PRODUCTS,
        'country_codes': PLAID_COUNTRY_CODES,
        'language': "en",
        'redirect_uri': None,
      }
    )
    return response
```


```python
token = create_link_token()
token
```

### Use Naas callback to get the plaid OAuth token


```python
cb_url = naas.callback.add()
```

### Select Bank connection


```python
uid = uuid.uuid4().hex
iframe = """
<head>
  <script src="https://cdn.plaid.com/link/v2/stable/link-initialize.js"></script>
</head>
<script>
const handler_{uid} = Plaid.create({
  token: '{GENERATED_LINK_TOKEN}',
  onSuccess: (public_token, metadata) => {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", "{CALLBACK_URL}", true);
        xhr.setRequestHeader('Content-Type', 'application/json');
        xhr.send(JSON.stringify({
            public_token: public_token
        }));
  }
});
handler_{uid}.open();
</script>
"""
iframe = iframe.replace('{uid}', uid)
iframe = iframe.replace('{CALLBACK_URL}', cb_url.get('url'))
iframe = iframe.replace('{GENERATED_LINK_TOKEN}', token.get('link_token'))
IPython.core.display.display(IPython.core.display.HTML(iframe))
```

### Get back plaid token


```python
cb_data = naas.callback.get(cb_url.get('uuid'))
cb_data = json.loads(cb_data)
public_token = cb_data.get("public_token")
public_token
```

## Output

### Exange token 


```python
exchange_response = client.Item.public_token.exchange(public_token)
access_token = exchange_response['access_token']
item_id = exchange_response['item_id']
```

### Show account list


```python
response = client.Accounts.balance.get(access_token)
accounts = response['accounts']
accounts_df = pd.DataFrame.from_records(accounts)
accounts_df
```

### Save as csv


```python
accounts_df.to_csv('accounts.csv')
```

#### If you need more data check the api doc 
https://plaid.com/docs/
