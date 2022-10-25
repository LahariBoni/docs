<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Twitter/Twitter_Send_posts_stats_to_Notion.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Twitter+-+Send+posts+stats+to+Notion:+Error+short+description">🚨 Bug report</a>

**Tags:** #twitter #post #comments #naas_drivers #snippet #content #notion

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maixmejublou)

This notebook will synchronize your Twitter posts stats with a Notion database.

## Input

### Import libraries


```python
import naas
from naas_drivers import notion
import re
import regex
from numpy import inf
try:
    import emoji
except:
    ! pip install --user emoji
    import emoji
```

### Setup Twitter


```python
TWITTER_CONSUMER_KEY = naas.secret.get('TWITTER_CONSUMER_KEY') or 'YourCredential'
TWITTER_CONSUMER_SECRET = naas.secret.get('TWITTER_CONSUMER_SECRET') or 'YourCredential'

TWITTER_BEARER_TOKEN = naas.secret.get('TWITTER_BEARER_TOKEN') or 'YourCredential'


TWITTER_ACCESS_TOKEN = naas.secret.get('TWITTER_ACCESS_TOKEN') or 'YourCredential'
TWITTER_ACCESS_TOKEN_SECRET = naas.secret.get('TWITTER_ACCESS_TOKEN_SECRET') or 'YourCredential'
```

### Setup Notion


```python
# Notion token
NOTION_TOKEN = naas.secret.get('NOTION_TOKEN') or 'YourNotionToken'

# Notion database url
notion_database_url = "https://www.notion.so/naas-official/ed622cae89e045249c464a08dc818876?v=989e444993d3421c8712e6e6b2d60810"
```

## Model

### Create a naas driver for Twitter


```python
import tweepy
import pandas as pd
import json
from typing import List
import datetime
import pydash

tweet_fields = ["author_id,created_at,source,public_metrics"]
tweet_personal_fields = ["author_id,created_at,source,public_metrics,non_public_metrics,organic_metrics"]


class Twitter:
    
    # Authenticate as an app.
    __bearer_token : str
    
    # Authenticate as a user.
    __consumer_key : str
    __consumer_secret : str
    __access_token : str
    __access_token_secret : str
    
    # Twitter v2 auth
    __app_client : tweepy.Client
    __user_client : tweepy.Client
    
    __me : pd.Series
    
    def connect(self, bearer_token:str, consumer_key:str, consumer_secret:str, access_token:str, access_token_secret:str) -> "Twitter":
        self.__bearer_token = bearer_token
        
        self.__app_client = tweepy.Client(
            bearer_token=self.__bearer_token
        )
        
        self.__consumer_key = consumer_key
        self.__consumer_secret = consumer_secret
        self.__access_token = access_token
        self.__access_token_secret = access_token_secret
        
        
        self.__user_client = tweepy.Client(
            consumer_key=consumer_key,
            consumer_secret=consumer_secret,
            access_token=access_token,
            access_token_secret=access_token_secret
        )
        
        self.__me = self.get_me()
        
        return self
    
    @property
    def app_client(self):
        return self.__app_client
    
    @property
    def user_client(self):
        return self.__user_client
    
    def get_user(self, username:str) -> pd.Series:
        users = self.__app_client.get_users(usernames=[username])
        if users is None:
            return None
        
        return pd.Series(users.data[0].data)
    
    def get_me(self):
        me = self.__user_client.get_me()
        if me is None:
            return None
        return pd.Series(me.data.data)
    
    def get_my_tweets(self, **kwargs):
        return self.get_users_tweets(self.__me.id, **kwargs)


    def get_users_tweets(self, user_id:str, tweet_count=10, tweet_fields: List[str] = tweet_fields, start_time = datetime.datetime.now() - datetime.timedelta(days=30), end_time=datetime.datetime.now()) -> pd.DataFrame:
        should_stop = False
        tweets_array = []
        next_token = None
        
        while len(tweets_array) < tweet_count and should_stop is False:
            tweets_left_to_fetch = tweet_count - len(tweets_array)
            
            if tweets_left_to_fetch > 100:
                max_results = 100
            elif tweets_left_to_fetch < 5:
                max_results = 5
            else:
                max_results = tweets_left_to_fetch
            
        
            tweets = self.__app_client.get_users_tweets(
                id=user_id,
                max_results=max_results,
                start_time=start_time,
                end_time=end_time,
                pagination_token=next_token,
            )
            next_token = pydash.get(tweets, 'meta.next_token', None)
            if next_token is None:
                should_stop = True

            is_own_tweets = user_id == self.__me.id

            for tweet in tweets.data:
                tweet_id = tweet.id


                if is_own_tweets is True:
                    rich_tweet_response = self.__user_client.get_tweet(tweet_id, tweet_fields=tweet_personal_fields, user_auth=True)
                    if len(rich_tweet_response.errors):
                        rich_tweet_response = self.__user_client.get_tweet(tweet_id, tweet_fields=tweet_fields, user_auth=True)
                else:
                    rich_tweet_response = self.__app_client.get_tweet(tweet_id, tweet_fields=tweet_fields, user_auth=False)


                rtd = rich_tweet_response.data

                tweets_array.append({
                    "id": rtd.id,
                    "url": f'https://twitter.com/{self.__me.username}/status/{rtd.id}',
                    "created_at": rtd.created_at,
                    "author_id": rtd.author_id,
                    "author_name": self.__me['name'],
                    "author_username": self.__me.username,
                    "text": rtd.text,
                    "public_retweet_count": pydash.get(rtd, 'public_metrics.retweet_count', -1),
                    "public_reply_count": pydash.get(rtd, 'public_metrics.reply_count', -1),
                    "public_like_count": pydash.get(rtd, 'public_metrics.like_count', -1),
                    "public_quote_count": pydash.get(rtd, 'public_metrics.quote_count', -1),
                    "organic_retweet_count": pydash.get(rtd, 'organic_metrics.retweet_count', -1),
                    "organic_reply_count": pydash.get(rtd, 'organic_metrics.reply_count', -1),
                    "organic_like_count": pydash.get(rtd, 'organic_metrics.like_count', -1),
                    "organic_quote_count": pydash.get(rtd, 'organic_metrics.quote_count', -1),
                    "non_public_user_profile_clicks": pydash.get(rtd, 'non_public_metrics.user_profile_clicks', -1),
                    "non_public_impression_count": pydash.get(rtd, 'non_public_metrics.impression_count', -1),
                })
        
        return pd.DataFrame(tweets_array).astype({
            "public_retweet_count": int, 
            "public_reply_count": int,
            "public_like_count": int,
            "public_quote_count": int,
            "organic_retweet_count": int,
            "organic_reply_count": int,
            "organic_like_count": int,
            "organic_quote_count": int,
            "non_public_user_profile_clicks": int,
            "non_public_impression_count": int
        })
        
    
twitter = Twitter()
```

### Connect Twitter driver


```python
twitter.connect(TWITTER_BEARER_TOKEN, TWITTER_CONSUMER_KEY, TWITTER_CONSUMER_SECRET, TWITTER_ACCESS_TOKEN, TWITTER_ACCESS_TOKEN_SECRET)
```

### Get our user


```python
user = twitter.get_me()
user
```

### Get our tweets


```python
tweets = twitter.get_my_tweets()
tweets
```

### Compute reach


```python
tweets['REACH'] = (tweets['public_retweet_count'] + tweets['public_reply_count'] + tweets['public_like_count'] + tweets['public_quote_count'] + tweets['non_public_user_profile_clicks']) / tweets['non_public_impression_count']
tweets = tweets.round({'REACH': 4})

tweets = tweets.fillna({
    "REACH": -1
})

tweets['REACH'] = tweets['REACH'].replace(inf, -1)
tweets['REACH'] = tweets['REACH'].apply(lambda x: 0 if x < 0 else x)
tweets['REACH']
```

### Connect Notion Driver


```python
notion.connect(NOTION_TOKEN)
```

### Get Notion database


```python
db = notion.database.query(notion_database_url, query={})
```

### Helper function to quickly get page from database


```python
def get_page_with_matching_property(db, property_name, property_value):
    return pydash.find(db, lambda x: str(x.properties[property_name]) == property_value)
```

### Helper function to extract emojis


```python
def get_emojis(text):
    emoji_list = []
    data = regex.findall(r"\X", text)
    for word in data:
        if any(char in emoji.UNICODE_EMOJI["en"] for char in word):
            emoji_list.append(word)
    return emoji_list

def get_tags(text):
        tags = []
        tags_list = re.findall("#[^#| ]+[a-zA-Z0-9]", text)
        for i in range(0, len(tags_list)):
            tag = tags_list[i]
            check_tag = True
            for t in tag:
                if not t.isalpha() and not t.isnumeric() and t != "#":
                    check_tag = False
                if check_tag is False:
                    break
            if check_tag is False:
                tag = tag.rsplit(t)[0]
            tags.append(tag)
        return tags
```

### Iterate over tweets and upsert page in Notion database


```python
page_updated = 0
page_created = 0
page_created_list = []

for _, tweet in tweets.iterrows():
    content_url = tweet.url
    notion_page = get_page_with_matching_property(db, 'Content URL', content_url)
    
    if notion_page is None:
        notion_page = notion.page.create(database_id=notion_database_url, title=tweet.text)
        page_created += 1
        page_created_list.append(notion_page.url)
        print(f'✅ New notion page create for tweet: {content_url}')
    else:
        page_updated += 1
        print(f'⚙️ Updating page {notion_page.url} for tweet {content_url}')
    
    notion_page.link('Content URL', content_url)
    notion_page.number('Engagment score', tweet['REACH']) # Typo here but it was already there in the database.
    notion_page.number('Engagement score', tweet['REACH'])
    notion_page.number('Views', tweet['non_public_impression_count'])
    notion_page.number('Likes', tweet['public_like_count'])
    notion_page.number('Comments', tweet['public_reply_count'])
    notion_page.date("Publication Date", str(tweet['created_at']))
    
    emojis_array = get_emojis(tweet['text'])
    notion_page.rich_text("Emojis", ' ,'.join(emojis_array))
    notion_page.number("Nb emojis", len(emojis_array))
    
    tags_array = get_tags(tweet['text'])
    notion_page.rich_text("Tags", ' '.join(tags_array))
    notion_page.number("Nb tags", len(tags_array))
    
    notion_page.select("Status", "Published ✨")
    notion_page.select("Platform", "Twitter")
    notion_page.select("Content type", "Text")
    notion_page.select("Author", tweet['author_name'])

    notion_page.update()
    print(f'✅ Page for tweet {content_url} updated!')
```

## Output

### Display results


```python
page_created_template = "\n\n".join(page_created_list)

print(f'''
✅ Execution completed!

Number of page created: {page_created}
Number of page updated: {page_updated}

Page created:

{page_created_template}

''')
```
