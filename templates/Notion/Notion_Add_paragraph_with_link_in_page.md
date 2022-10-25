<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Add_paragraph_with_link_in_page.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Add+paragraph+with+link+in+page:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #productivity #naas_drivers #block #paragraph #link #snippet #operations

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

This notebook shows how to add a paragraph with link in a Notion page

## Input

### Import libraries


```python
from naas_drivers import notion 
from naas_drivers.tools.notion import Link
```

### Setup Notion
<a href='https://docs.naas.ai/drivers/notion'>How to get your Notion integration token ?</a>


```python
# Enter Token API
token = "*****"

# Enter Database URL
page_url = "https://www.notion.so/naas-official/Daily-med-03952fcb93c045bba519a7564a64045e"

# Link
link = "https://app.naas.ai/"
```

## Model

### Get page


```python
page = notion.connect(token).page.get(page_url)
```

### Add new paragraph with link


```python
res = page.paragraph("Go to Naas")
res.paragraph.text[0].href = link
res.paragraph.text[0].text.link = Link(link)
```

## Output

### Update page in Notion


```python
page.update()
```
