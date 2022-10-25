<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_sentiment_emotion_irony_offensiveness_from_comments.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+sentiment+emotion+irony+offensiveness+from+comments:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #nlp #transformers #ai #post #comments #naas_drivers #content #snippet #dataframe

**Author:** [Nikolaj Groeneweg](https://www.linkedin.com/in/njgroene/)

## About this template

### What it does

This notebook gets all the comments on a LinkedIn post, and performs sentiment analysis, emotion classification and some semantic analysis on them. 
It classifies each comment and returns the following information:

- is the comment positive, negative or neutral?
- is the comment ironic?
- is the comment offensive?
- does the comment express joy, optimism, anger or sadness?

### References

This template is based on the following work :

F. Barbieri, J. Camacho-Collados, L. Neves and L.E. Anke (2020), *TweetEval: Unified Benchmark and Comparative Evaluation for Tweet Classification*, CoRR abs/2010.12421. Full paper : https://arxiv.org/abs/2010.12421. Official github : https://github.com/cardiffnlp/tweeteval

All credit goes to the above authors, any mistakes are on the author of this template. 
  

### Disclaimers

The machine learning models used in this template were trained on datasets of tweets.<br>Details can be found here : https://github.com/cardiffnlp/tweeteval/blob/main/README.md. 

These models may be expected to work well on shorter comments, but their output will become less reliable as the length of the text increases. 

The "emotion" classification performs rather unpredictably on very short comments. For many neutral comments it defaults to "joy", there being neutral category). It is most useful to identify and filter for negative emotions (sadness) and to identify interesting commments (optimism).

In general, caution is recommended when integrating these classifications in any automated decision pipeline.

## Input

### Import libraries


```python
!pip install protobuf==3.20.1 --user
```


```python
from naas_drivers import linkedin
from transformers import pipeline
from transformers import AutoModelForSequenceClassification
from transformers import TFAutoModelForSequenceClassification
from transformers import AutoTokenizer
import numpy as np
from scipy.special import softmax
import csv
import urllib.request
import os.path
import naas
```

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn credentials
LI_AT = None or naas.secret.get("LI_AT") # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = None or naas.secret.get("JSESSIONID")  # EXAMPLE ajax:8379907400220387585

# Enter post URL
POST_URL = "ENTER_YOUR_POST_URL_HERE"
```

## Model

Get the post comments and return them in a dataframe. <br> Colums are added for classifier output.<br>

**Available columns after classification :**
- PROFILE_ID
- PROFILE_URL
- PUBLIC_ID
- FIRSTNAME
- LASTNAME
- FULLNAME
- OCCUPATION
- PROFILE_PICTURE
- BACKGROUND_PICTURE
- PROFILE_TYPE
- TEXT
- SENTIMENT
- SENTIMENT_SCORE
- IRONY
- IRONY_SCORE
- OFFENSIVE
- OFFENSIVE_SCORE
- EMOTION
- EMOTION_SCORE
- CREATED_TIME
- LANGUAGE
- DISTANCE
- COMMENTS
- LIKES
- POST_URL
- DATE_EXTRACT


```python
df = linkedin.connect(LI_AT, JSESSIONID).post.get_comments(POST_URL)

# add columns for classification output
df.insert(loc=11, column='SENTIMENT', value=None)
df.insert(loc=12, column='SENTIMENT_SCORE', value=None)
df.insert(loc=13, column='IRONY', value=None)
df.insert(loc=14, column='IRONY_SCORE', value=None)
df.insert(loc=15, column='OFFENSIVE', value=None)
df.insert(loc=16, column='OFFENSIVE_SCORE', value=None)
df.insert(loc=17, column='EMOTION', value=None)
df.insert(loc=18, column='EMOTION_SCORE', value=None)
```


```python
def preprocess(text):
    """Preprocess text to be classified
    Replaces user-tags and URLs with neutral token
    """
    new_text = []
    for t in text.split(" "):
        t = '@user' if t.startswith('@') and len(t) > 1 else t
        t = 'http' if t.startswith('http') else t
        new_text.append(t)
    return " ".join(new_text)

def classify(text, task, tokenizers, models, task_labels):
    """Classifies text using task classifier
    with the corresponding tokenizer and model
    :return: dictionary with winning label and corresponding score
    """
    text = preprocess(text)
    tokenizer = tokenizers[task]
    model = models[task]
    labels = task_labels[task]
    encoded_input = tokenizer(text, return_tensors='pt')
    output = model(**encoded_input)
    scores = output[0][0].detach().numpy()
    scores = softmax(scores)
    ranking = np.argsort(scores)
    ranking = ranking[::-1]
    idx = ranking[0]
    label = str(labels[idx])
    score =np.round(float(scores[idx]), 4)
    return {"label":label, "score":score}
```


```python
# selected subset of available tasks
tasks = ["sentiment", "emotion", "irony", "offensive"]
# these labels are slightly modified to improve readibility
labels = {"sentiment":['negative', 'neutral', 'positive'], "emotion":['anger', 'joy', 'optimism', 'sadness'], "irony":['not-ironic', 'ironic'], "offensive":['not-offensive', 'offensive']}
# models and tokenizers will be loaded from huggingface
models = {}
tokenizers = {}

# perform each of the classifications and enrich dataframe
for task in tasks:
    MODEL = f"cardiffnlp/twitter-roberta-base-{task}"
    # on first run, tokenizer and model are loaded from hugging face
    tokenizer = AutoTokenizer.from_pretrained(MODEL)
    model = AutoModelForSequenceClassification.from_pretrained(MODEL)  
    
    # save tokenizer and models to local disk
    tokenizer.save_pretrained(MODEL)
    model.save_pretrained(MODEL)
    
    models[task] = model
    tokenizers[task] = tokenizer
    
    # execution time is not a concern, so we can just use .apply() to apply classifier
    result = df["TEXT"].apply(classify, args=(task,tokenizers, models, labels))
    
    # keep only winning label and score, inspect result to see full classifier output
    df[str.upper(task)] = [d['label'] for d in result]
    df[str.upper(task)+"_SCORE"] = [d['score'] for d in result]
    
df.head(5)
```

## Output

### Display result


```python
# shows only text and classification output
for index, row in df.iterrows():
    print(f"{row['TEXT']}\n\t{row['SENTIMENT']}({row['SENTIMENT_SCORE']})\n\t{row['IRONY']}({row['IRONY_SCORE']})\n\t{row['OFFENSIVE']}({row['OFFENSIVE_SCORE']})\n\t{row['EMOTION']}({row['EMOTION_SCORE']})\n\t\n\n")
```
