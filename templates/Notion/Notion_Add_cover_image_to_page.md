<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Add_cover_image_to_page.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a>

**Tags:** #notion #productivity #naas_drivers #operations #snippet

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

This notebook add cover image to page with external url

## Input

### Import libraries


```python
from naas_drivers import notion 
```

### Setup Notion
- [Get your Notion integration token](https://docs.naas.ai/drivers/notion)
- Share integration with your database / page


```python
# Enter Token API
NOTION_TOKEN = "ENTER_YOUR_NOTION_TOKEN_HERE" # EXAMPLE: "secret_xxxxxxxxxxxxxxxxxx"

# Enter Page URL
PAGE_URL = "ENTER_YOUR_PAGE_URL_HERE"  # EXAMPLE: "https://www.notion.so/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Enter icon URL
IMAGE_URL = "ENTER_YOUR_IMAGE_URL_HERE"  # EXAMPLE: "https://XXXXXXXXXXXX.png"
```

## Model

### Get page


```python
page = notion.connect(NOTION_TOKEN).page.get(PAGE_URL)
```

### Add icon image to page


```python
notion.client.pages.update(page_id=page.id, cover={'type': 'external', 'external': {'url': IMAGE_URL}})
```

## Output


```python
page.update()
```