<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YouTube/YouTube_Send_video_stats_to_Notion.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YouTube+-+Send+video+stats+to+Notion:+Error+short+description">🚨 Bug report</a>

**Tags:** #youtube #video #statistics #naas_drivers #content #snippet #dataframe

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

This notebook will synchronize your YouTube videos stats with a Notion database.

## Input

### Import library


```python
import naas
from naas_drivers import youtube, notion
import pandas as pd
import pydash
import re
import regex
try:
    import emoji
except:
    ! pip install --user emoji
    import emoji
```

### Variables

To know how to generate a YouTube api key you can [watch this video](https://www.youtube.com/watch?v=ltdJOX_DVtE).

<a href='https://docs.naas.ai/drivers/notion'>How to get your Notion integration token ?</a>


```python
# Youtube api Key
YOUTUBE_API_KEY = naas.secret.get('YOUTUBE_API_KEY') or 'YourYoutubeApiKey'

# Channel ID
channel_url = "https://www.youtube.com/channel/UCKKG5hzjXXU_rRdHHWQ8JHQ"

# Notion token
NOTION_TOKEN = naas.secret.get('NOTION_TOKEN') or 'YourNotionToken'

# Notion database url
notion_database_url = "https://www.notion.so/naas-official/ed622cae89e045249c464a08dc818876?v=989e444993d3421c8712e6e6b2d60810"
```

### Scheduling

Lets run this notebook every hour.


```python
#naas.scheduler.add(cron="0 * * * *")
```

## Model

### Connect YouTube driver


```python
youtube.connect(YOUTUBE_API_KEY)
```

### Get all uploads


```python
df_uploads = youtube.channel.get_uploads(channel_url=channel_url)
df_uploads
```

### Enrich DataFrame with each video statistics


```python
all_video_stats = None

for _, upload in df_uploads.iterrows():
    video_id = upload['VIDEO_ID']
    df_video_stats = youtube.video.get_statistics(f'https://www.youtube.com/watch?v={video_id}')
    if all_video_stats is None:
        all_video_stats = df_video_stats
    else:
        all_video_stats = pd.concat([all_video_stats, df_video_stats])
        
all_video_stats['VIDEO_ID'] = all_video_stats['ID']
df_uploads = df_uploads.merge(all_video_stats, on='VIDEO_ID')
df_uploads
```

### Compute reach


```python
df_uploads = df_uploads.astype({
    'COMMENTCOUNT': int, 
    'LIKECOUNT': int,
    'VIEWCOUNT': int
})
df_uploads['REACH'] = ((df_uploads['COMMENTCOUNT'] + df_uploads['LIKECOUNT'])) / df_uploads['VIEWCOUNT']
df_uploads = df_uploads.round({'REACH': 4})
df_uploads
```

### Connect Notion driver


```python
notion.connect(NOTION_TOKEN)
```

### Get notion database


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

### Iterate over videos and upsert page in Notion database


```python
page_updated = 0
page_created = 0
page_created_list = []

for _, video in df_uploads.iterrows():
    content_url = f"https://www.youtube.com/watch?v={video['VIDEO_ID']}"
    notion_page = get_page_with_matching_property(db, 'Content URL', content_url)
    if notion_page is None:
        notion_page = notion.page.create(database_id=notion_database_url, title=video['VIDEO_TITLE'])
        page_created += 1
        page_created_list.append(notion_page.url)
        print(f'✅ New notion page create for video: {content_url}')
    else:
        page_updated += 1
        print(f'⚙️ Updating page {notion_page.url} for video {content_url}')
    
    notion_page.link('Content URL', content_url)
    notion_page.number('Engagment score', video['REACH']) # Typo here but it was already there in the database.
    notion_page.number('Engagement score', video['REACH'])
    notion_page.number('Views', video['VIEWCOUNT'])
    notion_page.number('Likes', video['LIKECOUNT'])
    notion_page.number('Comments', video['COMMENTCOUNT'])
    notion_page.date("Publication Date", video['VIDEO_PUBLISHEDAT'])
    
    emojis_array = get_emojis(video['VIDEO_TITLE'] + video['VIDEO_DESCRIPTION'])
    notion_page.rich_text("Emojis", ' ,'.join(emojis_array))
    notion_page.number("Nb emojis", len(emojis_array))
    
    tags_array = get_tags(video['VIDEO_TITLE'] + video['VIDEO_DESCRIPTION'])
    notion_page.rich_text("Tags", ' '.join(tags_array))
    notion_page.number("Nb tags", len(tags_array))
    
    notion_page.select("Status", "Published ✨")
    notion_page.select("Platform", "YouTube")
    notion_page.select("Content type", "Video")
    
    notion_page.update()
    print(f'✅ Page for video {content_url} updated!')
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
