<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Remoteok/Remoteok_Post_daily_jobs_on_slack.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Remoteok+-+Post+daily+jobs+on+slack:+Error+short+description">🚨 Bug report</a>

**Tags:** #remoteok #jobs #slack #gsheet #naas_drivers #automation #opendata #text

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

## Input

### Import libraries


```python
import pandas as pd
import requests
from datetime import datetime
import time
from naas_drivers import gsheet, slack
import naas
```

### Setup slack channel configuration


```python
SLACK_TOKEN = "xoxb-1481042297777-3085654341191-xxxxxxxxxxxxxxxxxxxxxxxxx"
SLACK_CHANNEL = "05_jobs"
```

### Setup sheet log data

For the driver to fetch the contents of your google sheet, you need to share it with the service account linked with Naas first.
naas-share@naas-gsheets.iam.gserviceaccount.com


```python
spreadsheet_id = "1EBefhkbmqaXMZLRCiafabf6xxxxxxxxxxxxxxxxxxx"
sheet_name = "REMOTEOK_POSTS"
```

### Setup Remoteok

### Setting the parameters 


```python
categories = ['machine learning',
              'data science',
              'nlp',
              'deep learning',
              'computer vision',
              'data',
              'natural language processing',
              'data engineer']
date_from  = -10 ### this is 10 days from now => must be negative
```

### Set the Scheduler


```python
naas.scheduler.add(recurrence="0 9 * * *")
# # naas.scheduler.delete() # Uncomment this line to delete your scheduler if needed
```

## Model

### Get the sheet log of jobs


```python
try:
    df_jobs_log = gsheet.connect(spreadsheet_id).get(sheet_name=sheet_name)
except KeyError as e:
    print('Gsheet is empty!!')
    df_jobs_log = pd.DataFrame()
```

### Get jobs from RemoteOk


```python
REMOTEOK_API = "https://remoteok.com/api"
REMOTEOK_DATETIME = "%Y-%m-%dT%H:%M:%S"
NAAS_DATETIME = "%Y-%m-%d %H:%M:%S"

def get_jobs(remoteok_url, categories):
    df = pd.DataFrame()
    headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36',
        }
    index=0
    for tag in categories:
        url = remoteok_url + f"?tag={tag}"
        res = requests.get(url, headers=headers)
        try:
            res.raise_for_status()
        except requests.HTTPError as e:
            return e
        
        job_details = res.json()
        
        if len(job_details)==1:
            continue
        else:
            for idx, job in enumerate(job_details):
                if idx!=0:
                    date = job['date'].split('+')[0]
                    publication_time = datetime.strptime(date, REMOTEOK_DATETIME).timestamp()
                    required_time = time.time() + date_from* 24 * 60 * 60  ### time in seconds
                    
                    if publication_time >= required_time:
                        df.loc[index, 'URL'] = job.get('url')
                        df.loc[index, 'TITLE'] = job.get('position')
                        df.loc[index, 'COMPANY'] = job.get('company')
                        df.loc[index, 'TAGS'] = ", ".join(job.get('tags'))
                        df.loc[index, 'LOCATION'] = job.get('location')
                        df.loc[index, 'PUBLICATION_DATE'] = datetime.fromtimestamp(publication_time).strftime(NAAS_DATETIME)
                        index+=1
    
    df = df.drop_duplicates(subset = 'URL', keep='first')
    df = df.sort_values(by='PUBLICATION_DATE', ascending=False)
    return df

df_jobs = get_jobs(REMOTEOK_API, categories)
df_jobs.head()
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

### Send all job links to the slack channel


```python
if len(df_new_jobs) > 0:
    for _, row in df_new_jobs.iterrows():
        url = row.URL
        slack.connect(SLACK_TOKEN).send(SLACK_CHANNEL, f"<{url}>")
else:
    print("Nothing to be published in Slack !")
```
