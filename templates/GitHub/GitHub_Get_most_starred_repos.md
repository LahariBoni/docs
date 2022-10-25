<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Get_most_starred_repos.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Get+most+starred+repos:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #repos #stars #snippet

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190)

This notebook get the most starred repos in GitHub and return a dataframe with:
- repo_id
- name
- url
- stars
- forks
- issues_open
- created_at
- updated_at
- topics
- description

## Input

### Import libraries


```python
import requests
import pandas as pd
import plotly.express as px
import naas
```

### Setup Variables

* The Github search API provides up to 1,000 results for each search.

* Please visit [this link](https://docs.github.com/en/rest/search#about-the-search-api) to know more about Github search limitation.


```python
# Query number of repositories with stars greater than the given threshold
threshold = 500 #provides list of repos with stars greater than 500

# Setup how many top repository results are to be shown
top_n = 250

# if you want to fetch all the repository results with the
# given threshold instead of top_n number, then put in top_n value to 'all'

# Github token
GITHUB_TOKEN = None or naas.secret.get('GITHUB_TOKEN')
```

## Model

### Get most starred repos


```python
def fetch_results(top_n, threshold, token):
    URL = f"https://api.github.com/search/repositories?q=stars:%3E{threshold}&sort=stars"
    headers = {"Authorization": f"token {token}"}
    df = pd.DataFrame()
    cnt, page= 0,1
    
    while True:
        params = {
                    "state": "open",
                    "per_page": "100",
                    "page": page,
                }
        res = requests.get(URL, headers = headers, params = params)
        try:
            res.raise_for_status()
        except requests.HTTPError as e:
            if "422 Client Error: Unprocessable Entity for url:" in str(e):
                print('Github Search API limit reached!')
                print('Collecting the search results')
                break
        res_json = res.json()

        for r in res_json['items']:
            df.loc[cnt, 'repo_id'] = r['id']
            df.loc[cnt, 'name'], df.loc[cnt, 'url'] = r['name'], r['html_url']
            df.loc[cnt,'stars'], df.loc[cnt, 'forks'], df.loc[cnt, 'issues_open'] = r['watchers'], r['forks'], r['open_issues']
            df.loc[cnt,'created_at'], df.loc[cnt, 'updated_at'] = r['created_at'], r['updated_at']
            if len(r['topics']):
                df.loc[cnt, 'topics'] = ",".join(r['topics'])
            else:
                df.loc[cnt,'topics'] = 'None'

            if r['description']:
                df.loc[cnt,'description'] = r['description']
            else:
                df.loc[cnt,'description'] = 'None'

            cnt+=1
            if cnt == top_n:
                break
        
        if cnt == top_n:
            break
        
        page+=1
        
    df.stars, df.forks, df.issues_open, df.repo_id = df.stars.astype('int'), df.forks.astype('int'), df.issues_open.astype('int'), df.repo_id.astype('int')
    return df

df_results = fetch_results(top_n, threshold, GITHUB_TOKEN)
df_results.shape
```

## Output

### Display result


```python
df_results.head(10)
```
