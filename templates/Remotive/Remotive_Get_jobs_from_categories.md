<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Remotive/Remotive_Get_jobs_from_categories.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Remotive+-+Get+jobs+from+categories:+Error+short+description">🚨 Bug report</a>

**Tags:** #remotive #jobs #csv #snippet #opendata #dataframe

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

With this notebook, you will be able to get jobs offer from Remotive:
- **URL:** Job offer url.
- **TITLE:** Job title.
- **COMPANY:** Company name.
- **PUBLICATION_DATE:** Date of publication.

## Input

### Import libraries


```python
import pandas as pd
import requests
import time
from datetime import datetime
```

### Setup Remotive

#### Get categories from Remotive


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
df_categories
```

#### Enter your parameters


```python
categories = ['data'] # Pick the list of categories in columns "slug"
date_from = - 10 # Choose date difference in days from now => must be negative
```

### Variables


```python
csv_output = "REMOTIVE_JOBS.csv"
```

## Model

### Get all jobs posted after timestamp_date

All jobs posted after the date from will be fetched.<br>
In summary, we can set the value, in seconds, of 'search_data_from' to fetch all jobs posted since this duration


```python
REMOTIVE_DATETIME = "%Y-%m-%dT%H:%M:%S"
NAAS_DATETIME = "%Y-%m-%d %H:%M:%S"

def get_remotive_jobs_since(jobs, date):
    ret = []
    for job in jobs:
        publication_date = datetime.strptime(job['publication_date'], REMOTIVE_DATETIME).timestamp()
        if publication_date > date:
            ret.append({
                'URL': job['url'],
                'TITLE': job['title'],
                'COMPANY': job['company_name'],
                'PUBLICATION_DATE': datetime.fromtimestamp(publication_date).strftime(NAAS_DATETIME)
            })
    return ret

def get_category_jobs_since(category, date, limit):
    url = f"https://remotive.io/api/remote-jobs?category={category}&limit={limit}"
    res = requests.get(url)
    if res.json()['jobs']:
        publication_date = datetime.strptime(res.json()['jobs'][-1]['publication_date'], REMOTIVE_DATETIME).timestamp()
        if len(res.json()['jobs']) < limit or date > publication_date:
            print(f"Jobs from catgory {category} fetched ✅")
            return get_remotive_jobs_since(res.json()['jobs'], date)
        else:
            return get_category_jobs_since(category, date, limit + 5)
    return []

def get_jobs_since(categories: list,
                   date_from: int):
    if date_from >= 0:
        return("'date_from' must be negative. Please update your parameter.")
    # Transform datefrom int to
    search_jobs_from = date_from * 24 * 60 * 60   # days in seconds
    timestamp_date = time.time() + search_jobs_from

    jobs = []
    for category in categories:
        jobs += get_category_jobs_since(category, timestamp_date, 5)
    print(f'- All job since {datetime.fromtimestamp(timestamp_date)} have been fetched:', len(jobs))
    return pd.DataFrame(jobs)

df_jobs = get_jobs_since(categories, date_from=date_from)
df_jobs.head(5)
```

## Output

### Save dataframe in csv


```python
df_jobs.to_csv(csv_output, index=False)
```
