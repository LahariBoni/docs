<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/TikTok/TikTok_Get_user_stats.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=TikTok+-+Get+user+stats:+Error+short+description">🚨 Bug report</a>

**Tags:** #tiktok #user #stats #snippet #content

**Author:** [Alok Chilka](https://www.linkedin.com/in/calok64/)

## Input

### Import libraries


```python
try:
    from TikTokAPI import TikTokAPI
except:
    !pip install --user PyTikTokAPI
    from TikTokAPI import TikTokAPI
import nest_asyncio
import pandas as pd
```


```python
nest_asyncio.apply()
```

### Setup your TikTok


### How to get cookies ?

- "s_v_web_id and "tt_webid"

While logged into your tiktok account, Click F12 to open developer console in your browser. Navigate to path Storage -> Cookies -> https://www.tiktok.com and on right hand side there are the parameters marked which you will need.

![image.png](attachment:a631afda-67e0-45f6-b1ad-9d80a832fe1f.png)



```python
# Cookies
cookie = {
  "s_v_web_id": "verify_l0ecjehXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "tt_webid": "1%7CaSy-x8YGNmB_l9qsXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}

# Username
username = "tiktester04"
```

### Create connection object


```python
api = TikTokAPI(cookie=cookie)
```

## Model

### Get user stats


```python
def get_user_stats(username):
    #Get user details
    '''
    User detail fields : video count, follower count, following count, heart count
    '''

    user_obj = api.getUserByName(username)
    user_video_count = user_obj["userInfo"]["stats"]["videoCount"]
    user_follower_count = user_obj["userInfo"]["stats"]["followerCount"]
    user_following_count = user_obj["userInfo"]["stats"]["followingCount"]
    user_heart_count = user_obj["userInfo"]["stats"]["heartCount"]

    user_stats = {
        "user_video_count": user_video_count,
        "user_follower_count": user_follower_count,
        "user_following_count": user_following_count,
        "user_heart_count": user_heart_count

    }
    return user_stats
```

## Output

### Get user stats


```python
df_stats = get_user_stats(username)
df_stats
```
