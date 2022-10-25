<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YouTube/YouTube_Get_uploads_from_channel.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YouTube+-+Get+uploads+from+channel:+Error+short+description">🚨 Bug report</a>

**Tags:** #youtube #channel #videos #naas_drivers #content #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import library


```python
import naas
from naas_drivers import youtube
```

### Variables

To know how to generate a YouTube api key you can [watch this video](https://www.youtube.com/watch?v=ltdJOX_DVtE).


```python
# Youtube api Key
YOUTUBE_API_KEY = naas.secret.get('YOUTUBE_API_KEY')

# Channel ID
channel_url = "https://www.youtube.com/channel/UCKKG5hzjXXU_rRdHHWQ8JHQ"
```

## Model


```python
df_uploads = youtube.connect(YOUTUBE_API_KEY).channel.get_uploads(channel_url)
```

## Output

### Display results


```python
df_uploads
```
