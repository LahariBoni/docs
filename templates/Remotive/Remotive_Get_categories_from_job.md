<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Remotive/Remotive_Get_categories_from_job.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a>

**Tags:** #remotive #categories #snippet

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

## Input

### Imports


```python
import pandas as pd
import requests
```

## Model

### Get categories from Remotive


```python
def get_remotejob_categories():
    req_url = f"https://remotive.io/api/remote-jobs/categories"
    res = requests.get(req_url)
    try:
        res.raise_for_status()
    except requests.HTTPError as e:
        return e
    res_json = res.json()
    
    # Get categories
    jobs = res_json.get('jobs')
    return pd.DataFrame(jobs)

df_categories = get_remotejob_categories()
```

## Output

### Display result


```python
df_categories
```
