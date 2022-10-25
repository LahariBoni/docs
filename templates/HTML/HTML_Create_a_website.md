<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HTML/HTML_Create_a_website.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HTML+-+Create+a+website:+Error+short+description">🚨 Bug report</a>

**Tags:** #html #css #website #page #landing #custom #snippet #marketing

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

The objective of this notebook is to create an end-to-end website in 5min. 

## Input 

### Import libraries


```python
from urllib.request import urlopen
from IPython.display import IFrame
import naas 
```

## Model 

### Get example


```python
html = urlopen("http://www.example.com/").read().decode('utf-8')
print(html)
```

### Save file on your file system
Click right to open + edit the file downloaded 


```python
html_file = open("site.html","w")
html_file.write(html)
html_file.close()
```

### Learn about HTML with this Cheat Sheet


```python
IFrame("https://web.stanford.edu/group/csp/cs21/htmlcheatsheet.pdf", width=900, height=600)
```

### Manually change the content of the file to make it your own. 

Use Google search to go further in the customization (I recommend using https://stackoverflow.com/ + https://www.w3schools.com/html/)

## Output

Use Naas asset formula to generate a shareable URL.

Nb: if you want to use your own domain name, we will cover that in another version of this template.
Contact us → hello@naas.ai


```python
naas.asset.add("site.html", {"inline": True})
```
