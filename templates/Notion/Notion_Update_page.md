<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Update_page.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Update+page:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #productivity #naas_drivers #operations #snippet

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

This notebook shows how to use Naas Notion driver to update a page (properties + content) inside a database.

## Input

### Import libraries


```python
from naas_drivers import notion 
```

### Input variables
<a href='https://docs.naas.ai/drivers/notion'>How to get your Notion integration token ?</a>


```python
# Enter Token API
token = "*****"

# Enter Database URL
page_url = "https://www.notion.so/naas-official/Daily-med-03952fcb93c045bba519a7564a64045e"
```

## Model

### Get page
Get your page content and returns a dataframe with name of column, type and value.


```python
page = notion.connect(token).page.get(page_url)
page
```

### Update page properties 
Properties are associated with the database. If you put a page type that is not currently present, it will create it.


```python
page.title("Name","Page updated")
page.rich_text("Text","Ceci est toto")
page.number("Number", 42)
page.select("Select","Value3") 
page.multi_select("Muti Select",["Value1","Value2","Value3"])
page.date("Date","2021-10-03T17:01:26") #Follow ISO 8601 format
page.people("People", ["6e3bab71-beeb-484b-af99-ea30fdef4773"]) #list of ID of users
page.checkbox("Checkbox", False)
page.email("Email","jeremy@naas.ai")
page.phone_number("Phone number","+33 6 21 83 11 12")
page.update()
```

### Update page blocks 
Blocks are the content of the page.


```python
page.heading_1("Heading 1")
page.heading_2("Heading 2")
page.heading_3("Heading 3")
page.paragraph("Paragraph")
page.numbered_list_item("This is first")
page.to_do("Need this to be done")
page.embed("https://docs.google.com/spreadsheets/*************")
page.video("https://www.youtube.com/watch?v=8AsMAc4VFJs")
page.image("https://landen.imgix.net/jtci2pxwjczr/assets/5ice39g4.png")
page.code("pip install naas")
page.equation("e=mc2")
page.update()
```

## Output


```python
page
```
