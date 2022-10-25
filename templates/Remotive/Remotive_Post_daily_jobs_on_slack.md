<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Remotive/Remotive_Post_daily_jobs_on_slack.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Remotive+-+Post+daily+jobs+on+slack:+Error+short+description">🚨 Bug report</a>

**Tags:** #remotive #jobs #slack #gsheet #naas_drivers #automation #opendata #text

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

## Input

### Import libraries


```python
import pandas as pd
from bs4 import BeautifulSoup
import requests
from datetime import datetime
import time
from naas_drivers import gsheet, slack
import naas
```

### Setup slack channel configuration


```python
SLACK_TOKEN = "xoxb-1481042297777-3085654341191-xxxxxxxxxxxxxxxxxxxxxxxxx"
SLACK_CHANNEL = "05_work"
```

### Setup sheet log data


```python
spreadsheet_id = "1EBefhkbmqaXMZLRCiafabf6xxxxxxxxxxxxxxxxxxx"
sheet_name = "SLACK_CHANNEL_POSTS"
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

### Set the Scheduler


```python
naas.scheduler.add(recurrence="0 9 * * *")
# # naas.scheduler.delete() # Uncomment this line to delete your scheduler if needed
```

## Model

### Get the sheet log of jobs


```python
df_jobs_log = gsheet.connect(spreadsheet_id).get(sheet_name=sheet_name)
df_jobs_log
```

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
    print(f'- All job since {datetime.fromtimestamp(timestamp_date)} have been fetched -')
    return pd.DataFrame(jobs)

df_jobs = get_jobs_since(categories, date_from=date_from)
df_jobs
```

### Remove duplicate jobs


```python
def remove_duplicates(df1, df2):
    # Get jobs log
    jobs_log = df1.URL.unique()
    
    # Exclude jobs already log from jobs
    df2 = df2[~df2.URL.isin(jobs_log)]
    return df2.sort_values(by="PUBLICATION_DATE")

df_new_jobs = remove_duplicates(df_jobs_log, df_jobs)
df_new_jobs
```

## Output

### Add new jobs on the sheet log


```python
gsheet.connect(spreadsheet_id).send(sheet_name=sheet_name,
                                    data=df_new_jobs,
                                    append=True)
```

### Send all jobs link to the slack channel


```python
if len(df_new_jobs) > 0:
    for _, row in df_new_jobs.iterrows():
        url = row.URL
        slack.connect(SLACK_TOKEN).send(SLACK_CHANNEL, f"<{url}>")
else:
    print("Nothing to published in Slack !")
```
