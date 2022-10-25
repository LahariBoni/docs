<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Drive/Google_Drive_Download_file.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Drive+-+Download+file:+Error+short+description">🚨 Bug report</a>

**Tags:** #googledrive #snippet #operations #naas

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

Pre-requisite : change file access rights to make sure Naas can access ("Allow anyone")
If you need more specific user rights, just drop us an email hello@naas.ai

### Import library


```python
try:
    import gdown
except:
    !pip install gdown
    import gdown
```

## Model

### Url of the file and the name of the output


```python
url = 'https://drive.google.com/uc?id=1-3UlYEPKgL4E197umEh6d58jWknodSm5'
output = 'naas_happy_hour.mp4'
```

## Output

### Download the file


```python
gdown.download(url, output, quiet=False)
```
