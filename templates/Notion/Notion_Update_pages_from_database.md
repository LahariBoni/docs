<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Update_pages_from_database.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Update+pages+from+database:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #productivity #naas_drivers #operations #snippet

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

## Input

### Import library


```python
from naas_drivers import notion
```

### Setup Notion
<a href='https://docs.naas.ai/drivers/notion'>How to get your Notion integration token ?</a>


```python
# Enter Token API
notion_token = "*****"

# Enter Database URL
database_url = "https://www.notion.so/********"
```

## Model

### Get pages from Notion DB


```python
database_id = database_url.split("/")[-1].split("?v=")[0]
pages = notion.connect(notion_token).database.query(database_id, query={})
print("📊 Pages in Notion DB:", len(pages))
```

## Output

### Update pages


```python
for page in pages:
    print(page)
#     page.title("Name","Page updated")
#     page.rich_text("Text","Ceci est toto")
#     page.number("Number", 42)
#     page.select("Select","Value3") 
#     page.multi_select("Muti Select",["Value1","Value2","Value3"])
#     page.date("Date","2021-10-03T17:01:26") #Follow ISO 8601 format
#     page.people("People", ["6e3bab71-beeb-484b-af99-ea30fdef4773"]) #list of ID of users
#     page.checkbox("Checkbox", False)
#     page.email("Email","jeremy@naas.ai")
    page.update()
    print(f"✅ Page updated in Notion.")
```
