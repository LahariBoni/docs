<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Update_metrics_from_posts_in_Notion_content_calendar.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Update+metrics+from+posts+in+Notion+content+calendar:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #profile #post #feed #naas_drivers #notion #automation #analytics #naas #scheduler #content #plotly #html #csv #image

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

With this notebook, you can update your content calendar in Notion with all metrics from any profile posts in LinkedIn:
- **Publication date:** When the post has been published.
- **Name:** First sentence of post.
- **Author:** Name of post author.
- **Content type:** Content type of post (Image, Text, Video, Article, Polls).
- **Platform:** "LinkedIn" by default.
- **Status:** "Published ✨" by default.
- **Engagment score:** Ratio between views and likes/comments (It will be at 0 if you are not the author of the post)
- **Views:** Amount of people who saw the content (Only available on your profile)
- **Likes:** Amount of people who pushed the like (or other reaction) button
- **Comments:** Amount of people who wrote something in the comment section.
- **Shares:** Amount of people who shared the content.
- **Profile mention:** People mentioned in post.  
- **Company mention:** Companies mentioned in post.
- **Nb Tags:** Number of hashtags.
- **Tags:** List of the hashtags.  
- **Nb emojis:** Number of emojis.
- **Emojis:** List of emojis.
- **Nb links:** Number of links.
- **Links:** Links used in posts.
- **Nb characters:** Number of characters in post.  
- **Content URL:** URL of the content 
- **Last refresh:** Date of last refresh

## Input

### Import libraries
Here below is the list of tools needed.


```python
import naas
from naas_drivers import linkedin, notion, emailbuilder
from datetime import datetime
import pandas as pd
import plotly.express as px
import os
import requests
```

### Setup LinkedIn
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
# LinkedIn cookies
LI_AT = "ENTER_YOUR_COOKIE_HERE" # EXAMPLE : "AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2"
JSESSIONID = "ENTER_YOUR_JSESSIONID_HERE" # EXAMPLE : "ajax:8379907400220387585"

# LinkedIn profile url
PROFILE_URL = "ENTER_YOUR_LINKEDIN_PROFILE_URL_HERE" # EXAMPLE "https://www.linkedin.com/in/myprofile/"
```

### Setup Notion
- Configure naas integration
- Duplicate <a href="https://naas-official.notion.site/724fec443b134f288b356001bb1543bd?v=c82a8005a5bf4862b7c967a9689aa799">content calendar template</a>
- Share integration on the template


```python
# Notion parameters
NOTION_TOKEN = "ENTER_YOUR_NOTION_TOKEN_HERE" # EXAMPLE : "secret_eaLtxxxxxxxzuBPQvParsFxxxxxxx"
NOTION_DATABASE_URL = "ENTER_YOUR_NOTION_DATABASE_URL_HERE" # EXAMPLE : "https://www.notion.so/naas-official/fc64df2aae7f4796963d14edec816xxxxx"
```

### Setup Outputs
Create CSV to store your posts stats.<br>
PS: This CSV could be used in others LinkedIn templates.


```python
# Custom path of your CSV with the profile URL
profile_id = PROFILE_URL.split("https://www.linkedin.com/in/")[-1].split("/")[0]
csv_output = f"LINKEDIN_POSTS_{profile_id}.csv"
```

### Setup Naas scheduler
Schedule your notebook with the naas scheduler feature


```python
# the default settings below will make the notebook run everyday at 8:00
# for information on changing this setting, please check https://crontab.guru/ for information on the required CRON syntax 
naas.scheduler.add(cron="0 8 * * *")

# to de-schedule this notebook, simply run the following command: 
# naas.scheduler.delete()
```

## Model

### Get your posts from CSV
All your posts will be stored in CSV.


```python
def read_csv(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_posts = read_csv(csv_output)
df_posts
```

### Get updated posts
This function will get the last X posts from LinkedIn API (X = number of set in variable "NO_POSTS_RETRIEVED")<br>
PS: On the first execution all posts will be retrieved.


```python
def get_last_posts(df_posts,
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
                return df_new
    else:
        df_new = linkedin.connect(LI_AT, JSESSIONID).profile.get_posts_feed(PROFILE_URL,
                                                                            limit=-1)

    # Concat, save database in CSV and dependency in production
    df = pd.concat([df_new, df_posts]).drop_duplicates(key, keep="first")
    df["DATE_EXTRACT"] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    df.to_csv(csv_output, index=False)
    naas.dependency.add(csv_output)

    # Return only last post retrieved
    return df_new.reset_index(drop=True)

df_update = get_last_posts(df_posts,
                           csv_output)
df_update
```

## Output

### Update posts in Notion


```python
def create_polls_graph(uid, title, data):
    # Create dataframe
    df = pd.melt(pd.DataFrame([data]))
    df = df.sort_values(by="value")
    voters = df.value.sum()
    
    # Create fig
    fig = px.bar(df,
                 y="variable",
                 x="value",
                 orientation='h',
                 title=f"{title}<br><span style='font-size: 13px;'>Total amount of votes: {voters}</span>",
                 text="value",
                 labels={"variable": "Poll options",
                         "value": "Poll results"}
                 )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        plot_bgcolor="#ffffff",
        width=600,
        height=400,
        font=dict(family="Arial", size=14, color="black"),
        paper_bgcolor="white",
        xaxis_title=None,
        xaxis_showticklabels=False,
        yaxis_title=None,
        margin_pad=10,
    )
    fig.write_html(f"{uid}.html")
    asset = naas.asset.add(f"{uid}.html", params={"inline": True})
    return asset

def update_poll_graph(row):
    poll_graph = None
    
    # Add polls
    poll_id = row.POLL_ID
    poll_question = row.POLL_QUESTION
    poll_results = row.POLL_RESULTS
    if poll_id and poll_question and poll_results:
        poll_graph = create_polls_graph(poll_id, poll_question, poll_results)
    return poll_graph
```


```python
def update_dynamic_properties(page, row):
    # Page properties : dynamic
    page.number("Engagment score", float(row.ENGAGEMENT_SCORE))
    page.number("Views", int(row.VIEWS))
    page.number("Likes", int(row.LIKES))
    page.number("Comments", int(row.COMMENTS))
    page.number("Shares", int(row.SHARES))
    return page
```


```python
def update_content_notion(df, database_url):
    # Decode database id
    database_id = database_url.split("/")[-1].split("?v=")[0]
    
    # Get pages from notion database
    pages = notion.connect(NOTION_TOKEN).database.query(database_id, query={})
    
    # Manage dataframe empty
    if len(df) == 0:
        print(f"🛑 Nothing to update in Notion.")
        return
    
    # Loop in data
    df.COMPANY_MENTION = df.COMPANY_MENTION.fillna("")
    df.PROFILE_MENTION = df.PROFILE_MENTION.fillna("")
    for i, row in df.iterrows():
        title = row.TITLE
        content_title = row.CONTENT_TITLE
        if title is None and content_title is not None:
            title = f"Repost - {content_title}"
        elif title is None and content_title is None:
            title = "Repost"
        post_url = row.POST_URL
        print(post_url)
        
        # Create or update page
        page_new = True
        for page in pages:
            page_temp = page.df()
            page_id = page_temp.loc[page_temp.Name == "Content URL", "Value"].values
            if page_id == post_url:
                page_new = False
                break
        try:
            if page_new:
                # Create new page in notion
                page = notion.Page.new(database_id=database_id).create()

                # Page properties : static
                page.date("Publication Date", row.PUBLISHED_DATE)
                page.title("Name", title)
                page.select("Content type", row.CONTENT)
                page.select("Platform", "LinkedIn")
                page.select("Status", "Published ✨")
                page.select("Author", row.AUTHOR_NAME)
                profile_mention = row.PROFILE_MENTION
                if profile_mention is not None:
                    if len(profile_mention) > 2:
                        page.rich_text("Profile mention", profile_mention)
                company_mention = row.COMPANY_MENTION
                if company_mention is not None:
                    if len(company_mention) > 2:
                        page.rich_text("Company mention", company_mention)
                page.number("Nb tags", int(row.TAGS_COUNT))
                tags = row.TAGS
                if tags is None:
                    tags = ""
                else:
                    if len(tags) < 2:
                        tags = ""
                page.rich_text("Tags", tags)
                page.number("Nb emojis", int(row.EMOJIS_COUNT))
                emojis = row.EMOJIS
                if emojis is None:
                    emojis = ""
                else:
                    if len(emojis) < 2:
                        emojis = ""
                page.rich_text("Emojis", emojis)
                page.number("Nb links", int(row.LINKS_COUNT))
                links = row.LINKS
                if links is not None:
                    if len(links) > 2:
                        page.link("Links", links)
                page.number("Nb characters", int(row.CHARACTER_COUNT))
                page.link("Content URL", post_url)
                
                # Page blocks text
                page.heading_1("Text")
                text = row.TEXT
                if text is not None:
                    split_text = text.split("\n")
                    for t in split_text:
                        page.paragraph(t)
                    
                # Page blocks content
                image_url = row.IMAGE_URL
                content_url = row.CONTENT_URL
                poll_question = row.POLL_QUESTION
                if image_url or content_title or content_url or poll_question:
                    page.heading_1("Content")
                
                # Add image in content section
                if image_url:
                    page.heading_2("Image")
                    page.paragraph(image_url)
                    
                # Add post in content section
                if content_title:
                    page.heading_2("Media")
                    page.heading_3(content_title)
                
                if content_url:
                    page.paragraph(content_url)
                
                # Add poll graph in content section
                if poll_question:
                    page.heading_3("Poll")
                    poll_graph = update_poll_graph(row)
                    if poll_graph:
                        page.embed(poll_graph)
                
                # Page properties : dynamic
                page = update_dynamic_properties(page, row)
                
                # Create page in Notion
                page.update()
                print(f"✅ Page '{title}' created in Notion.", '\n')
            else:
                # Update poll graph
                update_poll_graph(row)
                
                # Page properties : dynamic
                page = update_dynamic_properties(page, row)
                
                # Update page
                page.update()
                print(f"📈 Post stats updated in notion for page '{title}'.", '\n')
        except Exception as e:
            print(f"❌ Error creating page '{title}' in Notion", e)
            print(row)
```


```python
update_content_notion(df_update, NOTION_DATABASE_URL)
```
