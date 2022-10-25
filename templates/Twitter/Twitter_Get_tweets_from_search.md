<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Twitter/Twitter_Get_tweets_from_search.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Twitter+-+Get+tweets+from+search:+Error+short+description">🚨 Bug report</a>

**Tags:** #twitter #ifttt #naas_drivers #snippet #content #dataframe

**Author:** [Dineshkumar Sundaram](https://github.com/dineshh912)

## Input

### Import libraries


```python
import tweepy
import pandas as pd
```

### API Credentials


```python
consumer_key = "XXXXXXXXXXXXXXXXXX"
consumer_secret = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```

### How to generate API Keys?

[Twitter API Documentation](https://developer.twitter.com/en/docs/getting-started)

## Model

### Authentification


```python

try:
    auth = tweepy.AppAuthHandler(consumer_key, consumer_secret)
    api = tweepy.API(auth)
except BaseException as e:
    print(f"Authentication has been failed due to -{str(e)}")
```

### Fonctions


```python
def getTweets(search_words, date_since, numTweets):
    
    # Define a pandas dataframe to store the date:
    tweets_df = pd.DataFrame(columns = ['username', 'desc', 'location', 'following',
                                        'followers', 'totaltweets', 'usercreated', 'tweetcreated',
                                        'retweetcount', 'text', 'hashtags']
                                )

    # Collect tweets using the Cursor object
    # .Cursor() returns an object that you can iterate or loop over to access the data collected.
    tweets = tweepy.Cursor(api.search, q=search_words, lang="en", since=date_since, tweet_mode='extended').items(numTweets)
    # Store tweets into a python list
    tweet_list = [tweet for tweet in tweets]
    for tweet in tweet_list:
        username = tweet.user.screen_name
        desc = tweet.user.description
        location = tweet.user.location
        following = tweet.user.friends_count
        followers = tweet.user.followers_count
        totaltweets = tweet.user.statuses_count
        usercreated = tweet.user.created_at
        tweetcreated = tweet.created_at
        retweetcount = tweet.retweet_count
        hashtags = tweet.entities['hashtags']
        try:
            text = tweet.retweeted_status.full_text
        except AttributeError:
            text = tweet.full_text
        
        tweet_data = [username, desc, location, following, followers, totaltweets,
                        usercreated, tweetcreated, retweetcount, text, hashtags]
        
        tweets_df.loc[len(tweets_df)] = tweet_data
        
    
    return tweets_df
```

### Initialise these function attributes:


```python
search_words = "#jupyterlab OR #python OR #naas OR #naasai"
date_since = "2021-09-21"
numTweets = 50
```

## Output

### Get the tweets


```python
df = getTweets(search_words, date_since, numTweets)
```
