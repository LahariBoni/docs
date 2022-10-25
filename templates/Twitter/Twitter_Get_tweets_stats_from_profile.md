<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Twitter/Twitter_Get_tweets_stats_from_profile.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Twitter+-+Get+tweets+stats+from+profile:+Error+short+description">🚨 Bug report</a>

**Tags:** #twitter #tweets #scrap #snippet #content #dataframe

**Author:** [Tannia Dubon](https://www.linkedin.com/in/tanniadubon/)

## Input

### Import library


```python
import os
import re
import pandas as pd
```


```python
#install developer snscrape package via command line
os.system("pip3 install git+https://github.com/JustAnotherArchivist/snscrape.git")
```

### Variables


```python
#criteria for searching by username
username = "JupyterNaas"
tweet_count = 500
```

## Model 

### Scrap and save results in JSON 


```python
#search by username using command line
os.system("snscrape --jsonl --max-results {} twitter-search from:{} > user-tweets.json".format(tweet_count, username))
```

###  Read JSON


```python
# Reads the json generated from the CLI command above and creates a pandas dataframe
df = pd.read_json('user-tweets.json', lines=True, convert_dates=True, keep_default_dates=True)
df
```

### Clean dataframe to keep only necessary columns

- URL
- TITLE
- CONTENT
- HASTAGS
- DATE
- LIKES
- RETWEETS


```python
#copy dataframe 
df1 = df.copy()

#keep only the columns needed
df1 = df1[['url','content','hashtags','date','likeCount','retweetCount']]

#convert columns to upper case to follow naas df convention
df1.columns = df1.columns.str.upper()

#convert time to ISO format to follow naas date convention
df1.DATE = pd.to_datetime(df1.DATE).dt.strftime("%Y-%m-%d")

#clean HASHTAGS column to provide searchable items in columns
df1.HASHTAGS = df1.HASHTAGS.fillna("[]")
df1.HASHTAGS = df1.apply(lambda row: ", ".join(list(row.HASHTAGS)) if row.HASHTAGS != '[]' else "", axis=1)

#display results
df1
```

## Output

### Save to df


```python
df1.to_csv("tweets_from_URL.csv", index=False)
```
