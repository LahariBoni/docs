<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Neo/Neo_Get_currencies_live_prices.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Neo+-+Get+currencies+live+prices:+Error+short+description">🚨 Bug report</a>

**Tags:** #neo #bank #snippet #finance #csv

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Install packages


```python
!pip install requests
```

### Import libraries


```python
from pathlib import Path
import pandas as pd
import requests
```

## Model

### API configurations


```python
ccy_pair = 'EUR/CHF'
AUTH_HOST = "https://auth.getneo.com"
DATA_HOST = "https://data.getneo.com"
login_id = 'LOGIN'
api_key = 'PASSWORD'
bearer_token = None
```

### Output file config


```python
output_dir = Path("../data_output/")
output_file = f"{ccy_pair.replace('/', '')}.csv"
```

### Authenticate to get token


```python
if not bearer_token:
    response = requests.get(f"{AUTH_HOST}/api/v1/auth/login/", params={"login_id": login_id, "api_key": api_key})
    if response.status_code != 200:
        raise PermissionError(f"Failed to authenticate as {login_id}")
    print(f"Authenticated as {login_id}")

    bearer_token = response.headers["Authorization"]
```

### Get data with token


```python
response = requests.get(f"{DATA_HOST}/api/v1/prices/{ccy_pair}", headers={"Authorization": bearer_token})
if response.status_code != 200:
    bearer_token = None
    raise ValueError(f"Failed to retrieve {ccy_pair} prices")
```

## Output

### Display result


```python
prices_dict = response.json()
prices = pd.DataFrame(data=list(prices_dict.items()), columns=["DATE", "VALUE"])
oldest_date = prices["DATE"].min()
print(f"Closing price for {ccy_pair} on {oldest_date}: {prices_dict[oldest_date]}")
newest_date = prices["DATE"].max()
print(f"Closing price for {ccy_pair} on {newest_date}: {prices_dict[newest_date]}")
```

### Saving output to file


```python
prices.to_csv(output_dir / output_file, index=False)
```
