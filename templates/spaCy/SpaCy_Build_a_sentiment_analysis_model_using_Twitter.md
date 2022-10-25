<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/spaCy/SpaCy_Build_a_sentiment_analysis_model_using_Twitter.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=spaCy+-+SpaCy+Build+a+sentiment+analysis+model+using+Twitter:+Error+short+description">🚨 Bug report</a>

**Tags:** #twitter #spaCy #data #nlp #sentiment #classification

**Author:** [Tannia Dubon](https://www.linkedin.com/in/tanniadubon/)

Tweets are retrieved using the Twitter Developer Platform and spaCy models are used for natural language processing.

Binary classification is used to score the sentiment of the tweets; 1 designated as positive, 0 as negative and a .5 threshold. <br>

Use this Notebook to update the query and conduct your own research! 

## Input

### Install required packages as necessary


```python
%pip install --user requests pandas pyyaml datetime numpy datetime matplotlib wordcloud seaborn spacy
```


```python
import os
import requests
import pandas as pd
import json
import ast
import yaml

import numpy as np
from datetime import datetime, date

import matplotlib.pyplot as plt
import matplotlib as mpl
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator

import re
import seaborn as sns
import string
import warnings
import random
import spacy
from spacy.training import Example
from spacy.pipeline.textcat import DEFAULT_SINGLE_TEXTCAT_MODEL
```

### Setup Bearer Token to Use Twitter API

#### How to get a bearer token ?

bearer_token – the token used for authentication https://developer.twitter.com/en/docs/authentication/oauth-2-0/bearer-tokens



```python
BEARER_TOKEN = "..."
```

## Setup Twitter query


```python
# Twitter query to fetch tweets. Learn how to build one here: https://developer.twitter.com/en/docs/twitter-api/tweets/search/integrate/build-a-query
TWITTER_QUERY = '(Putin OR Lukashenka OR Russia OR "Vladimir Putin") -is:retweet lang:en'

# Number of tweets to fetch (max 100).
TWITTER_MAX_RESULTS = 100
```

## Model

### Retrieve data from Twitter
Enter keywords for query in the create_url() function. 
See https://tinyurl.com/2j5phrhu for instructions on customizing your query syntax


```python
def create_url(query, max_results=100):
    tweet_fields = "tweet.fields=created_at,public_metrics,context_annotations,text,possibly_sensitive,geo"
    url = "https://api.twitter.com/2/tweets/search/recent?max_results={}&query={}&{}".format(max_results, query, tweet_fields)
    return url

def create_headers(bearer_token):
    headers = {"Authorization": "Bearer {}".format(bearer_token)}
    return headers

def connect_to_endpoint(url, headers):
    response = requests.request("GET", url, headers=headers)
    if response.status_code != 200:
        raise Exception(response.status_code, response.text)
    return response.json()
```

### Download tweets and store them in a json file.



```python
url= create_url(TWITTER_QUERY, TWITTER_MAX_RESULTS)
bearer_token = BEARER_TOKEN
headers = create_headers(bearer_token)
json_response = connect_to_endpoint(url, headers)
json_response
```

#### Define functions to retrieve each tweet field of interest


```python
#Initialize Lists
id_data = []
date_data = [] 
rtwt_data = []
reply_data = []
text_data = []
```


```python
def get_id():
    for data in json_response["data"]:
        id_record = data["id"]
        id_data.append(id_record)

def get_created_at():
    for data in json_response["data"]:
        date_record = data["created_at"]
        date_data.append(date_record)

def retweet_count():
    for retweets in json_response["data"]:
        rtwt_count=retweets["public_metrics"]["retweet_count"] 
        rtwt_data.append(rtwt_count)

def reply_count():
    for reply in json_response["data"]:
        reply_count = reply["public_metrics"]["reply_count"]
        reply_data.append(reply_count)

        
def get_text():
    for data in json_response["data"]:
        text_record = data["text"]
        text_data.append(text_record)
        
get_id() 
get_created_at() 
retweet_count()
reply_count() 
get_text()
```


```python
def get_domain():
    d_id = []
    d_name = []
    d_desc = []
    d_twt_id = []
    
    for j in json_response["data"]:
        for i in j:
            if i == "context_annotations":
                for d in j["context_annotations"]:
                    d_id.append(d["domain"]["id"])
                    d_name.append(d["domain"]["name"])
                    d_desc.append(d["domain"]["description"])
                    d_twt_id.append(j["id"])
                    
    domain = {"id": d_id, "name": d_name, "desc": d_desc,"tweet id": d_twt_id}
    return domain


def get_entity():    
    e_id = []
    e_name = []
    e_desc = []
    e_twt_id = []

    for j in json_response["data"]:
        for i in j:
            if i == "context_annotations":
                for d in j["context_annotations"]:
                    e_id.append(d["entity"]["id"])
                    e_name.append(d["entity"]["name"])
                    e_twt_id.append(j["id"])
                
    entity = {"id": e_id, "name": e_name, "desc": e_desc,"tweet id": e_twt_id}
    return entity
```


```python
#inspect output
print(get_entity())
```


```python
print(get_domain())
```


```python
print(len(id_data), len(date_data), len(rtwt_data), len(reply_data), len(text_data))
```

### Save data to pandas dataframe and clean and format text


```python
#saved tweet fields doe not include entity or domain data
save_data = {"id": id_data, "date": date_data, "retweet count": rtwt_data, "reply count": reply_data, "text": text_data}
df = pd.DataFrame(save_data)
df.to_csv("pol_tweet_data.csv")
df
```


```python
def cleanup_text(file, rem_item):
    r = re.findall(rem_item, file)
    for i in r:
        file = re.sub(i, "", file)
    return file
```


```python
#remove handles
df["clean text"] = np.vectorize(cleanup_text)(df["text"], "@[\w]*")
df["clean text"].head()
```


```python
len(df["clean text"])
print(df["clean text"])
```

### Tokenize text in spaCy


```python
#convert to tokens
nlp = None
try:
    nlp = spacy.load("en_core_web_lg") #here you can load a model that you'd previously trained
except:
    ! python -m spacy download en_core_web_lg
    nlp = spacy.load("en_core_web_lg")
    
text = str(df["clean text"])
doc = nlp(text)
```


```python
tokens_list = [] 
for token in doc:
    tokens_list.append(token)

tokens_list
```


```python
#Review entities recognized
for ent in doc.ents:
    print(ent.text, ent.label_)
```


```python
#Add a category for entities of interest, if needed. 
from spacy.matcher import PhraseMatcher 
matcher = PhraseMatcher(nlp.vocab) 

#define politicians as entities 
terms = ["Putin", "Zelensky"] 
patterns = [nlp.make_doc(term) for term in terms] 
matcher.add("politiciansList", None, *patterns) 

matches = matcher(doc) 

#this prints out the spans where the instances are found and the entity identified
for mid, start, end in matches: 
    print(start, end, doc[start:end])
```

### Text Classification


```python
config = {
    "threshold": 0.5,
    "model": DEFAULT_SINGLE_TEXTCAT_MODEL }

textcat = nlp.add_pipe("textcat", config=config) 
```


```python
#create training data for your example consisting of examples of positive and negative sentiment
train_data = [("Helping refugees. This is what kindness looks like.", {"cats": {"POS": True}}),
              ("In this time of uncertainty, we have a clear way forward: Help Ukraine defend itself. Support the Ukrainian people. Hold Russia accountable.", {"cats": {"POS": True}}),
              ("Priests demand head of Ukrainian Orthodox Church Moscow Patriarchate be brought to church tribunal for position on war.", {"cats": {"POS": True}}),
              ("Mayor of the most northern village in Ukraine Hremiach Hanna Havrylina was released after yesterday’s prisoners’ swap.", {"cats": {"POS": True}}),
              ("Look at this female volunteer from Belarus fighting alongside Ukrainians.", {"cats": {"POS": True}}),
              ("Russian soldiers: They're animals... Humans don't behave like this. My parents told me about WW2 & the fascists didn't even do such things.", {"cats": {"NEG": True}}),
              ("All Russians are evil", {"cats": {"NEG": True}}),
              ("The West is pushing Ukraine toward a conflict.", {"cats": {"NEG": True}}),
              ("Cowards", {"cats": {"NEG": True}}),
              ("Russia’s deployment of combat forces is a mere repositioning of troops on its own territory.", {"cats": {"NEG": True}}),
              ("Ukraine and Ukrainian government officials are the aggressor in the Russia-Ukraine relationship.", {"cats": {"NEG": True}})] 
```


```python
textcat.add_label("POS")
textcat.add_label("NEG")
    
train_examples = [Example.from_dict(nlp.make_doc(text), label) for text,label in train_data] 
```


```python
textcat.initialize(lambda: train_examples, nlp=nlp)
```


```python
#Define training example

epochs = 20

#Disable other pipe components & define training loop to incorporate statistical information

with nlp.select_pipes(enable="textcat"):
    optimizer = nlp.resume_training() #Creates optimizer object
    for i in range(epochs):
        random.shuffle(train_data)
        for text, label in train_data:
            doc = nlp.make_doc(text)
            example = Example.from_dict(doc, label) 
            print(nlp.update([example], sgd=optimizer))
```


```python
#enter an example tweet to test results
doc2 = nlp("As Russia continues to commit horrific atrocities against the Ukrainian people, we must take additional steps to cut off")

print(doc2.cats)
```


```python
#enter another example
doc3 = nlp("One of the captured Russian soldiers who was sent by Putin to “denazify” Ukraine")
print(doc3.cats)
```


```python
#process each row in clean text column
df["nlp_proc"] = [nlp(i) for i in df["clean text"]]
```


```python
#save positive/negative predictions to cats column
df["cats"] = [i.cats for i in df["nlp_proc"]]
```


```python
#assign value of 1 to positive classification, 0 to negative
sc_val = []

for i in df["cats"]:
    if i["POS"] >= .5:
        sc_val.append(1)
    else:
        sc_val.append(0)
```


```python
#append classification score to dataframe
df["score"] = sc_val
```


```python
#check dataframe
df
```


```python
#print out tweet id, text and score = to review results
for index, i in enumerate(df["score"]):
    if i == 1:
        print(df["id"][index], df["clean text"][index], df["score"][index])
```

## Output

### Display wordcloud


```python
#wordcloud
wordcloud = WordCloud(stopwords = STOPWORDS, collocations=True).generate(str(tokens_list))

plt.imshow(wordcloud, interpolation="bilinear")
plt.axis("off")
plt.show()
```

### Plot positive vs negative tweets


```python
ax = df.score.value_counts().plot(kind="bar", colormap="Paired")
plt.show()
```

### Save model for future use


```python
from pathlib import Path
output_dir=Path("spaCy_models")

def save_model(output_dir):
    if output_dir is not None:
        output_dir = Path(output_dir)
        if not output_dir.exists():
            output_dir.mkdir()
        nlp.to_disk(output_dir)
        print("Saved model to", output_dir)
        
save_model(output_dir)
```


```python
### To load trained, custom model on new data use:
#nlp = spacy.load("spaCy_models")
```
