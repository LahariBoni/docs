<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_posts_engagements.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+posts+engagements:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #posts #interactions #metrics #analytics #automation #naas

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This template get profile or company posts engagements (Likes and comments).

Outputs will 3 databases:
- Likes
- Comments
- Engagements = Likes + Comments

<div class="alert alert-info" role="info" style="margin: 10px">
<b>Requirements:</b><br>
To run this notebook, you must have already run <b>LinkedIn_Get_profile_posts_stats.ipynb</b> or <b>LinkedIn_Get_company_posts_stats.ipynb</b> to get your post stats in CSV.<br>
</div>

## Input

### Import libraries


```python
import naas
import pandas as pd
from datetime import datetime
```

### Setup Variables


```python
# Input
# -> Rename input if needed
csv_input = "LINKEDIN_PROFILE_POSTS.csv" # CSV path with your posts stats generated with 'LinkedIn_Get_profile_posts_stats.ipynb' or 'LinkedIn_Get_company_posts_stats.ipynb'

# Outputs
# -> Rename outputs if needed
csv_engagements = "LINKEDIN_ENGAGEMENTS.csv"
csv_likes = "LINKEDIN_LIKES.csv"
csv_comments = "LINKEDIN_COMMENTS.csv"
```

### Setup Naas dependency


```python
naas.dependency.add()

#-> Uncomment the line below to remove your dependency
# naas.dependency.delete()
```

## Model

### Read CSV


```python
def read_csv(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df
```

### Get your posts
Get posts feed from CSV stored in your local (Returns empty if CSV does not exist)


```python
df_posts = read_csv(csv_input)
print("✅ Posts fetched:", len(df_posts))
df_posts.head(1)
```

### Get who likes your posts


```python
df_likes = read_csv(csv_likes)
print("✅ Likes fetched:", len(df_likes))
df_likes.head(1)
```

### Get who comments your posts


```python
df_comments = read_csv(csv_comments)
print("✅ Comments fetched:", len(df_comments))
df_comments.head(1)
```

## Output

### Update engagements


```python
def update_engagements(df_posts,
                       df_interaction,
                       interaction,
                       csv_output,
                       no_posts=7,
                       min_updated_time=60):
    # Init
    df_out = df_interaction.copy()
    
    # Get all interactions if dataframe init empty or not complete
    if len(df_posts) == 0:
        return pd.DataFrame()

    if len(df_out) > 0:
        if "DATE_EXTRACT" in df_out.columns:
            last_update_date = df_out["DATE_EXTRACT"].max()
            time_last_update = datetime.now() - datetime.strptime(last_update_date, "%Y-%m-%d %H:%M:%S")
            minute_last_update = time_last_update.total_seconds() / 60
            if minute_last_update > min_updated_time:
                df_posts = df_posts[:no_posts]
            else:
                print(f"🛑 Nothing to update. Last update done {int(minute_last_update)} minutes ago.")
                return df_out.reset_index(drop=True)
    else:
        df_posts["SCENARIO"] = pd.to_datetime(df_posts["PUBLISHED_DATE"].str[:-6]).dt.strftime("%Y-%m")
        df_posts = df_posts[df_posts["SCENARIO"] == datetime.now().strftime("%Y-%m")].reset_index(drop=True)
        
    # Loop on posts
    for index, row in df_posts.iterrows():
        df_update = pd.DataFrame()
        post_title = row.TITLE
        post_author = row.AUTHOR_NAME
        post_url = row.POST_URL
        post_date = row.PUBLISHED_DATE
        count_interactions = row[interaction]
        print(f"🔄 {index+1} - Update started on: '{post_title}' ({post_url})")
        
        # Get interactions from post URL
        if len(df_interaction) > 0:
            tmp_df = df_interaction[df_interaction.POST_URL == post_url]
            no_interactions = len(tmp_df)
            if count_interactions != no_interactions:
                print(f"--> {count_interactions} post interaction count vs {no_interactions} interactions.")
                df_update = get_engagements(interaction, post_url)
            else:
                print("--->🛑 Nothing to update.")
        else:
            df_update = get_interactions(interaction, post_url)
        
        # Concat dataframe and save dataframe in CSV
        if len(df_update) > 0:
            print(f"---> {len(df_update)} interactions fetched.")
            df_update['TITLE'] = post_title
            df_update['AUTHOR_NAME'] = post_author
            df_update['PUBLISHED_DATE'] = post_date
            keys = ["POST_URL", "PROFILE_ID"]
            if interaction == "COMMENTS":
                keys = ["POST_URL", "PROFILE_ID", "CREATED_TIME"]
            df_out = pd.concat([df_update, df_out]).drop_duplicates(keys, keep="first")
            output_path = df_out.to_csv(csv_output)
            
    # Add dependency in production
    print(f"✅ {len(df_out)} '{interaction}' fetched.")
    # Return all interactions
    return df_out.reset_index(drop=True)
```

### Update likes
It will update your last X posts like's from LinkedIn API.<br>
PS: On the first execution all posts like's from the current month will be retrieved.


```python
df_update_likes = update_engagements(df_posts,
                                     df_likes,
                                     "LIKES",
                                     csv_likes)
df_update_likes.head(1)
```

### Update comments
It will update your last X posts like's from LinkedIn API.<br>
PS: On the first execution all posts like's from the current month will be retrieved.


```python
df_update_comments = update_engagements(df_posts,
                                        df_comments,
                                        "COMMENTS",
                                        csv_comments)
df_update_comments.head(1)
```

### Create engagements database


```python
def create_engagements_db(df_likes, df_comments):
    if len(df_likes) == 0 and len(df_comments) == 0:
        return pd.DataFrame()
    
    # Init outputs
    df = pd.DataFrame()
    
    # Dataframe likes
    df_likes["REACTION"] = "LIKES"

    # Dataframe comments
    df_comments["REACTION"] = "COMMENTS"
    
    # Concat
    df = pd.concat([df_likes, df_comments]).fillna("Not defined").sort_values(by="PUBLISHED_DATE", ascending=False)
    
    # Cleaning
    to_keep = [
        "PROFILE_ID",
        "PROFILE_URL",
        "PUBLIC_ID",
        "FIRSTNAME",
        "LASTNAME",
        "FULLNAME",
        "OCCUPATION",
        "REACTION",
        "TEXT",
        "TITLE",
        "PUBLISHED_DATE",
        "AUTHOR_NAME",
        "POST_URL",
    ]
    df = df[to_keep]
    
    print(f"✅ {len(df)} interactions fetched.")
    return df.reset_index(drop=True)

df_engagements = create_engagements_db(df_update_likes, df_update_comments)
df_engagements.head(3)
```

### Save data engagements


```python
df_engagements.to_csv(csv_engagements)
```
