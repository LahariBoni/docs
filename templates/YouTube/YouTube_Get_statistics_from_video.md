<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YouTube/YouTube_Get_statistics_from_video.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YouTube+-+Get+statistics+from+video:+Error+short+description">🚨 Bug report</a>

**Tags:** #youtube #video #statistics #naas_drivers #content #snippet #dataframe

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
video_url = "https://www.youtube.com/watch?v=W8H57kam9kg&t=3685s"
```

## Model


```python
df_stats = youtube.connect(YOUTUBE_API_KEY).video.get_statistics(video_url)
```

## Output

### Display results


```python
df_stats
```
