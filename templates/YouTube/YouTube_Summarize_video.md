<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YouTube/YouTube_Summarize_video.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YouTube+-+Summarize+video:+Error+short+description">🚨 Bug report</a>

**Tags:** #youtube #transcript #video #npl #naas_drivers #content #snippet #text

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import library


```python
from naas_drivers import youtube
```

### Variables


```python
video_url = "https://www.youtube.com/watch?v=4ds2FDI_60g&list=PLseBeHeWM4DHat4s5W2OeetaPB3sN2oTL&index=7"
```

## Model

### Extract the transcript in JSON


```python
summary = youtube.transcript.summarize(video_url)
```

## Output

### Display result


```python
summary
```
