<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Twitter/Twitter_Get_user_data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Twitter+-+Get+user+data:+Error+short+description">🚨 Bug report</a>

**Tags:** #twitter #ifttt #naas_drivers #snippet #content #dataframe

**Author:** [Dineshkumar Sundaram](https://github.com/dineshh912)

## Input

### Import library


```python
import tweepy
import pandas as pd
```

### How to get API keys?

[Twitter API Documentation](https://developer.twitter.com/en/docs/getting-started)

### Variables


```python
# API Credentials
consumer_key = "XXXXXXXXXXXXXXXXXX"
consumer_secret = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```


```python
user_list = ["JupyterNaas", "Spotify", "ProjectJupyter"]
```

## Model

### Authentication


```python
try:
    auth = tweepy.AppAuthHandler(consumer_key, consumer_secret)
    api = tweepy.API(auth)
except BaseException as e:
    print(f"Authentication has been failed due to -{str(e)}")
```

### Below function will retrive only the information about the user.


```python
def getUserInfo(user_id):
    
    # Define a pandas dataframe to store the date:
    user_info_df = pd.DataFrame(columns = ['twitter_id', 'name', 'screen_name', 'description', 'tweet_count', 'friends_count',
                        'followers_count', 'favourites_count', 'verified', 'created_at']
                                )

    # Collect userinformation using get_user
    for user in user_id:
        info = api.get_user(user) # Get user information request
        
        twitter_id = info.id
        name = info.name
        screen_name = info.screen_name
        description = info.description
        tweet_count = info.statuses_count
        friends_count = info.friends_count
        followers_count = info.followers_count
        favourites_count = info.favourites_count
        verified = info.verified
        created_at = info.created_at
        
       
        user_info = [twitter_id, name, screen_name, description, tweet_count, friends_count,
                        followers_count, favourites_count, verified, created_at]
        
        user_info_df.loc[len(user_info_df)] = user_info
        
    
    return user_info_df
```

## Output

### Get user data


```python
df = getUserInfo(user_list)
```
