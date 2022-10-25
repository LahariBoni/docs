<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Remoteok/Remoteok_Get_jobs_from_categories.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Remoteok+-+Get+jobs+from+categories:+Error+short+description">🚨 Bug report</a>

**Tags:** #remoteok #jobs #csv #snippet #opendata #dataframe

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

With this notebook, you will be able to get jobs offer from Remoteok:
- **URL:** Job offer url.
- **TITLE:** Job title.
- **COMPANY:** Company name.
- **TAGS:** Tags link to job.
- **LOCATION:** Location link to job.
- **PUBLICATION_DATE:** Date of publication.

## Input

### Import libraries


```python
import pandas as pd
import requests
from datetime import datetime
import time
```

### Setup Remoteok


```python
categories = ['machine learning',
              'data science',
              'nlp',
              'deep learning',
              'computer vision',
              'data',
              'natural language processing',
              'data engineer']
date_from  = -30 ### this is 30 days from now => must be negative
```

### Variables


```python
csv_output = "REMOTIVE_JOBS.csv"
```

## Model

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
                        
    df = df.sort_values(by='PUBLICATION_DATE', ascending=False)
    return df

df_jobs = get_jobs(REMOTEOK_API, categories)
df_jobs.head(5)
```

## Output

### Save dataframe in csv


```python
df_jobs.to_csv(csv_output, index=False)
```
