<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Send_weekly_post_engagement_metrics_by_email.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Send+weekly+post+engagement+metrics+by+email:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #tool #posts #engagement #metrics #analytics #automation #email #naas #notification

**Author:** [Nikolaj Groeneweg](https://www.linkedin.com/in/njgroene/)

This notebook sets up a weekly email with your the most interesting LinkedIn post engagement metrics; making it easier to track your efforts to engage your LinkedIn audience. Please be sure to set all your personal information before running (Cookies, LinkedIn profile and email address).

## Input

### Import libraries


```python
from naas_drivers import linkedin
import naas
from dateutil.parser import parse
import matplotlib.pyplot as plt
try:
    import seaborn as sns
except:
    !pip install seaborn --user
    import seaborn as sns
import pandas as pd
from datetime import datetime, date
import random
import time
```

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn cookies
LI_AT = "ENTER_YOUR_COOKIE_HERE" # EXAMPLE : "AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2"
JSESSIONID = "ENTER_YOUR_JSESSIONID_HERE" # EXAMPLE : "ajax:8379907400220387585"

# LinkedIn profile url
PROFILE_URL = "ENTER_YOUR_LINKEDIN_PROFILE_HERE" # EXAMPLE "https://www.linkedin.com/in/myprofile/"

# The first execution all posts will be retrieved.
# Then, you can use the parameter below to setup the number of posts you want to retrieved from LinkedIn API everytime this notebook is run.
NO_POSTS_RETRIEVED = 10
```

### Setup Naas notification


```python
EMAIL_TO = "ENTER_RECIPIENT_EMAIL_HERE" # you will receive weekly summary at this email 
EMAIL_FROM = None # summary will have this email as sender. Only available for your naas email, otherwise you will receive this email from notification@naas.ai
EMAIL_SUBJECT = 'LinkedIn Metrics' # subject of your email
```

### Setup Outputs
Create CSV to store your posts feed 


```python
# Custom Path of your CSV with profile URL
profile_id = PROFILE_URL.split("https://www.linkedin.com/in/")[-1].split("/")[0]
csv_output = f"LINKEDIN_POSTS_{profile_id}.csv"
```

### Setup Naas scheduler


```python
# Change your remote timezone if needed. By default remote timezone is "Europe/Paris"
# naas.set_remote_timezone("Europe/Lisbon")

# the default settings below will make the notebook run at 12:00 each Saturday
# for information on changing this setting, please check https://crontab.guru/ for information on the required CRON syntax 
naas.scheduler.add(cron="0 12 * * 6")
```


```python
# this notebook will run each week until de-scheduled
# to de-schedule this notebook, simply run the following command: 
# naas.scheduler.delete()
```

## Model

### Get your posts from CSV
All your posts will be stored in CSV.


```python
def get_posts(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_posts = get_posts(csv_output)
df_posts
```

### Update last posts
It will get the last X posts from LinkedIn API (X = number of set in variable "NO_POSTS_RETRIEVED") and update it in your CSV.<br>
PS: On the first execution all posts will be retrieved.


```python
def update_last_posts(df_posts,
                      csv_output,
                      key="POST_URL",
                      no_posts=10,
                      min_updated_time=60):
    # Init output
    df_new = pd.DataFrame()
    
    # Init df posts is empty then return entire database
    if len(df_posts) > 0:
        if "DATE_EXTRACT" in df_posts.columns:
            last_update_date = df_posts["DATE_EXTRACT"].max()
            time_last_update = datetime.now() - datetime.strptime(last_update_date, "%Y-%m-%d %H:%M:%S")
            minute_last_update = time_last_update.total_seconds() / 60
            if minute_last_update > min_updated_time:
                # If df posts not empty get the last X posts (new and already existing)
                df_new = linkedin.connect(LI_AT, JSESSIONID).profile.get_posts_feed(PROFILE_URL,
                                                                                    limit=no_posts,
                                                                                    sleep=False)
            else:
                print(f"🛑 Nothing to update. Last update done {round(minute_last_update, 0)} minutes ago.")
    else:
        df_new = linkedin.connect(LI_AT, JSESSIONID).profile.get_posts_feed(PROFILE_URL,
                                                                            limit=-1)

    # Concat, save database in CSV and dependency in production
    df = pd.concat([df_new, df_posts]).drop_duplicates(key, keep="first")
    df["DATE_EXTRACT"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    df.to_csv(csv_output, index=False)
    naas.dependency.add(csv_output)

    # Return only last post retrieved
    return df.reset_index(drop=True)

df_posts = update_last_posts(df_posts,
                             csv_output,
                             no_posts=NO_POSTS_RETRIEVED)
df_posts
```

### Get post published this week


```python
def get_iso_year(row) :
    return parse(row.PUBLISHED_DATE).isocalendar()[0]

def get_iso_week(row) :
    return parse(row.PUBLISHED_DATE).isocalendar()[1]

def get_iso_day(row) :
    return parse(row.PUBLISHED_DATE).isoweekday()

def get_iso_day_string(row) :
    return datetime.strptime(str(parse(row.PUBLISHED_DATE).isoweekday()), "%d").strftime("%A")

# week of the year
df_posts['ISO_YEAR'] = df_posts.apply(get_iso_year, axis=1)
# week of the year
df_posts['ISO_WEEK'] = df_posts.apply(get_iso_week, axis=1)
# day of the year
df_posts['DAY'] = df_posts.apply(get_iso_day, axis=1)
# day of the week
df_posts['DAY OF WEEK'] = df_posts.apply(get_iso_day_string, axis=1)

# all posts made this week
df_this_week = df_posts[(df_posts['ISO_WEEK'] == date.today().isocalendar()[1]) &
                        (df_posts['ISO_YEAR'] == date.today().isocalendar()[0])].sort_values('DAY')
df_this_week.head()
```

### Send email if no post published this week


```python
# if we didn't post this week, send a simple email of encouragement and exit
if(df_this_week.empty):
    post = """\
    <h2>It looks like you took a week off from posting...</h2>
    <h4>Good for you, you'll crush it next week!</h4>"""
    naas.notification.send(email_to=EMAIL_TO,
                           subject=EMAIL_SUBJECT,
                           html=post,
                           email_from=EMAIL_FROM)
    raise SystemExit("No posts this week, basic email sent")
```

### Get engagement metrics


```python
likes = df_this_week.LIKES.sum()
views = df_this_week.VIEWS.sum()
shares = df_this_week.SHARES.sum()
comments = df_this_week.COMMENTS.sum()

# preview of what will be send by email:
print("This week's cumulative statistics:")
print("\n\t👀 Impressions\t", views, "\n\t👍 Likes\t", likes, "\n\t💬 Comments\t", comments, "\n\t⏩ Shares\t", shares)
```

### Information on top post


```python
top_post = df_this_week.sort_values("VIEWS", ascending=False).iloc[0]

# get information on most viewed post
top_post_text = top_post["TEXT"]
top_post_url = top_post["POST_URL"]
top_post_views = top_post["VIEWS"]
top_post_likes = top_post["LIKES"]
top_post_comments = top_post["COMMENTS"]
top_post_day = top_post["DAY OF WEEK"]
top_post_text = top_post_text.rstrip()[0:128] + " (...)"

# preview of what will be send by email:
print("✏️ Your best post this week got 👀 x", top_post_views, "and 👍 x", top_post_likes)
print("You posted it on", top_post_day,"\n")
print("This is what it said:\n\n", top_post_text)
print("See your post on LinkedIn : ", top_post_url)
```

### Weekly engagement plots


```python
sns.set_style("darkgrid")
fig, axs = plt.subplots(ncols=2)
fig.tight_layout()
sns.lineplot(x="DAY OF WEEK", y="VIEWS", data=df_this_week, ax=axs[0])
sns.lineplot(x="DAY OF WEEK", y="LIKES", data=df_this_week, palette="deep", ax=axs[1])

# save image to production as a public asset :
output_image = 'week.png'
fig.savefig(output_image)
link_image = naas.asset.add(output_image)
```

### This week's biggest fan


```python
def get_likes(df_posts):
    df_out = pd.DataFrame()
    for index, row in df_posts.iterrows():
        df = linkedin.connect(LI_AT, JSESSIONID).post.get_likes(row['POST_URL'])
        df_out = df_out.append(df)
        # aditional pause to protect your LinkedIn account 
        time.sleep(random.uniform(3, 6))
    return df_out

# count likes for each profile 
def count_likes(df):
    to_group = ["PROFILE_ID",
                "FIRSTNAME",
                "LASTNAME"]
    df2 = df.groupby(['PROFILE_ID']).size().sort_values(ascending=False).reset_index(name='LIKE_COUNT')
    return df2
```


```python
df_likes = get_likes(df_this_week)
df_counts = count_likes(df_likes)

top_fan = df_likes[df_likes["PROFILE_ID"] == df_counts["PROFILE_ID"][0]]

fan_slug = top_fan["PROFILE_ID"].iloc[0]
fan_first_name = top_fan["FIRSTNAME"].iloc[0]
fan_last_name = top_fan["LASTNAME"].iloc[0]
fan_name = f"{fan_first_name} {fan_last_name}"  
fan_likes = df_counts["LIKE_COUNT"][0]
fan_url = f"https://www.linkedin.com/in/{fan_slug}"

# preview of what will be send by email 
print("❤️ This week", fan_name, "was your biggest fan! They left you", fan_likes, "likes")
print("Go say hi on their profile:", fan_url)
```

### Create email content


```python
# You can edit the basic HTML below to change the look and feel of the email
# but be sure to pay attention to the variable names (indicated by $)
# so you don't unexpectedly break anything

html = """\
<h2>Here's your weekly LinkedIn report!</h2>
<h4>🚀 You did great this week !</h4>

&emsp;👀 $VIEWS impressions<br>
&emsp;👍 $LIKES likes<br>
&emsp;💬 $COMMENTS comments<br> 
&emsp;⏩ $SHARES shares<br>

<h4>❤️ This week <i>$FAN_NAME</i> was your biggest fan!</h4>
&emsp;They left you 👍 x $FAN_LIKES<br><br>
&emsp;<a href="$FAN_URL">Go say hi on their profile</a><br>

<h4>✏️ This week your best post got 👀 x $BEST_VIEWS and 👍 x $BEST_LIKES</h4>
&emsp;You posted it on $BEST_DAY, and this is what it said: <br>
<br>
&emsp;&emsp;<i>$BEST_TEXT</i>
<br><br>
&emsp;<a href="$BEST_URL">See your post on LinkedIn</a>

<h4>📈 Here, have some charts (we're out of cookies)</h4>
<img src="$CHART" style="width:640px; height:360px;" /><br>
"""
```


```python
html = html.replace("$VIEWS", str(views))
html = html.replace("$LIKES", str(likes))
html = html.replace("$COMMENTS", str(comments))
html = html.replace("$SHARES", str(shares))
html = html.replace("$FAN_NAME", str(fan_name))
html = html.replace("$FAN_LIKES", str(fan_likes))
html = html.replace("$FAN_URL", str(fan_url))
html = html.replace("$BEST_VIEWS", str(top_post_views))
html = html.replace("$BEST_LIKES", str(top_post_likes))
html = html.replace("$BEST_DAY", str(top_post_day))
html = html.replace("$BEST_TEXT", str(top_post_text))
html = html.replace("$BEST_URL", str(top_post_url))
html = html.replace("$CHART", str(link_image))
post = html

# preview final email
print(post)
```

## Output

### Send post engagement by email


```python
# sends the email
naas.notification.send(email_to=EMAIL_TO,
                       subject=EMAIL_SUBJECT,
                       html=post,
                       email_from=EMAIL_FROM)
```
