<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YouTube/YouTube_Download_video.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YouTube+-+Download+video:+Error+short+description">🚨 Bug report</a>

**Tags:** #youtube #download #video #content #snippet #naas

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Install packages 


```python
!pip install pytube3 --upgrade
```

### Import library


```python
from pytube import YouTube
```

### Variables


```python
youtube_video_url = 'https://www.youtube.com/watch?v=bleU_nrckr8'

yt_obj = YouTube(youtube_video_url)
```

## Model

### Paste Youtube URL


```python
for stream in yt_obj.streams:
    print(stream)
```

### Filter on MP4 Quality


```python
filters = yt_obj.streams.filter(progressive=True, file_extension='mp4')
 
for mp4_filter in filters:
    print(mp4_filter)
```


```python
filters = yt_obj.streams.filter(progressive=True, file_extension='mp4')
 
filters.get_highest_resolution()
filters.get_lowest_resolution()
```

## Output

### Download video on current directory


```python
filters.get_highest_resolution().download()
```
