<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Twitter/Twitter_Get_posts_stats.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Twitter+-+Get+posts+stats:+Error+short+description">🚨 Bug report</a>

**Tags:** #twitter #post #comments #naas_drivers #snippet #content

**Author:** [Alok Chilka](https://www.linkedin.com/in/calok64/)

## Input

### Import libraries


```python
!pip install --user tweepy
```


```python
from naas_drivers import linkedin, hubspot
import pandas as pd
import numpy as np
import naas
from datetime import datetime, timedelta
import requests
import json
import tweepy
```

### Setup your Twitter

### How to get token ?

bearer_token – the token used for authentication
https://developer.twitter.com/en/docs/authentication/oauth-2-0/bearer-tokens



```python
#Twitter cookies
bearer_token = 'AAAAAAAAAAAAAAAAAAAAAFGZZgEAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'

client = tweepy.Client(bearer_token=bearer_token)
user_name = "<YOUR_PREFERRED_USERNAME>"
users = client.get_users(usernames=[user_name])
user_id = users.data[0]["id"]

#set application type. This is used to distinguish data for application in master data model
APP_TYPE = "Twitter"

BASE_URL = "https://twitter.com/"
```

### Setup your email to receive notifications


```python
SEND_EMAIL_TO = "<YOUR_EMAIL_ID>"
```

## Model

### Get posts from Twitter feed


```python
#Get user tweets
tweets = client.get_users_tweets(id=user_id)
```

### Build tweet id list


```python
tweetIdList = []
for tweet in tweets.data:
    tweetIdList.append(str(tweet["id"]))
```


```python
tweetIdList
```

### Get stats from Twitter posts


```python
def get_post_stats(tweetIdList):
    twitterDF = pd.DataFrame()
    for tweetId in tweetIdList:

        #print("*********************************")
        #print("Tweet ID : ", tweetId)
        parent_tweet = client.get_tweet(id=tweetId, tweet_fields=["author_id","created_at","entities","in_reply_to_user_id",
                                                                        "referenced_tweets,source,public_metrics"])
        parent_user_name = user_name
        parent_Tweet_Title = ""
        #parent_tweet_id = reply_tweet.data["data"]["referenced_tweets"][0]["id"]
        parent_tweet_text = parent_tweet.data["text"]
        parent_created_at = parent_tweet.data["created_at"].strftime("%d-%m-%Y %H:%M:%S")
        parent_tweet_URL = BASE_URL+user_name+"/status/"+tweetId
        parent_public_metrics = parent_tweet.data["data"]["public_metrics"]
        parent_retweet_count = parent_public_metrics["retweet_count"]
        parent_reply_count = parent_public_metrics["reply_count"]
        parent_like_count = parent_public_metrics["like_count"]
        parent_quote_count = parent_public_metrics["quote_count"]
        parent_view_count = 0 #currently twitter has no ways to find out who actually viewed your post hence kept value = 0 to map the columns
        parent_comments_mentions = []

        retweeted_by_users = client.get_retweeters(id=tweetId)
        
        retweeted_by_username = []
        if retweeted_by_users.data != None:
            for retweet_user in retweeted_by_users.data:
                retweeted_by_username.append(retweet_user["username"])
        
        like_by_users = client.get_liking_users(id=tweetId)        
        liked_by_username = []
        if like_by_users.data != None:
            for like_user in like_by_users.data:
                liked_by_username.append(like_user["username"])
        
        mentions = client.get_users_mentions(id=user_id)
        
        comments = []
        if mentions.data != None:
            for mention in mentions.data:
                child_tweet = client.get_tweet(id=mention["id"], tweet_fields=["author_id", "in_reply_to_user_id","referenced_tweets,source,public_metrics,text"])
                in_reply_to_tweet_id = child_tweet.data["data"]["referenced_tweets"][0]["id"]

                if str(in_reply_to_tweet_id) == tweetId:
                    username = "@"+user_name
                    temp = str(mention["text"])
                    temp = temp.replace(username,"")
                    comments.append(temp)
        
        data=[[tweetId,parent_created_at,parent_user_name,parent_Tweet_Title,parent_tweet_text,parent_tweet_URL,parent_view_count,parent_reply_count,parent_like_count,parent_retweet_count,APP_TYPE]]
        df = pd.DataFrame(data,columns=["ACTIVITY_ID","PUBLISHED_DATE","AUTHOR_NAME","TITLE","TEXT","POST_URL","VIEWS","COMMENTS","LIKES","SHARES","APPLICATION_TYPE"
])
        twitterDF = twitterDF.append(df)

    return twitterDF
```

## Output

Available Columns / Data :

- Text
- Created At
- Public Metrics (Retweet Count, Reply Count,Like Count)
- List of users who retweeted 
- List of users who liked the tweet
- Comments with mentions

More details can be found at https://developer.twitter.com/en/docs/twitter-api/data-dictionary/object-model/user


```python
get_post_stats(tweetIdList)
```
