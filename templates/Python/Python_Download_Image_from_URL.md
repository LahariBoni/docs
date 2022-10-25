<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Download_Image_from_URL.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Download+Image+from+URL:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #image #url #naas #snippet

**Author:** [Abraham Israel](https://www.linkedin.com/in/abraham-israel/)

This template downloads an image from an URL and to display the results inside a notebook.

## Input

### Import libraries


```python
try:
    import wget
except:
    !pip install wget
    import wget
```

### Setup Variables


```python
image_url = "https://i0.wp.com/thenerddaily.com/wp-content/uploads/2018/08/Reasons-To-Watch-Anime.jpg"
```

## Model

### Download Image


```python
image_name = wget.download(image_url)
```

## Output

### Display result


```python
print("Image downloaded: /", image_name)

from IPython.display import Image
Image(filename=image_name) 
```


```python

```
