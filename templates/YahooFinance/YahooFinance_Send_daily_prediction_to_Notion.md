<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YahooFinance/YahooFinance_Send_daily_prediction_to_Notion.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YahooFinance+-+Send+daily+prediction+to+Notion:+Error+short+description">🚨 Bug report</a>

**Tags:** #yahoofinance #trading #markdown #prediction #plotly #slack #naas_drivers #scheduler #notification #asset #webhook #dependency #naas #investors #automation #analytics #html #image #notion

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

With this template, you can get any ticker available in [Yahoo finance](https://finance.yahoo.com/quote/TSLA/), add predictions and update a Notion DB.<br> 

## Input

### Import libraries


```python
import naas
from naas_drivers import prediction, yahoofinance, plotly, notion
from datetime import datetime
from naas_drivers.tools.notion import Link, BlockEmbed
import pytz
```

### Setup Yahoo Finance
👉 Here you can change the ticker and timeframe


```python
TICKER = "TSLA"
date_from = -100 # 1OO days max to feed the naas_driver for prediction
date_to = "today"
```

### Setup Prediction
👉 Here you can change the number of data points you want the prediction will be performed on


```python
DATA_POINT = 20
```

### Setup Notion
- [Get your Notion integration token](https://docs.naas.ai/drivers/notion)
- Share integration with your database


```python
# Credentials
NOTION_TOKEN = "YOUR_NOTION_TOKEN"
DATABASE_URL = "https://www.notion.so/naas-official/f42d6592949axxxxxxxxxxxxx"

# Setup your page title, this will be the key to update your data
PAGE_TITLE = "Tesla stock"
```

### Setup Assets


```python
NOW = datetime.now().strftime("%Y-%m-%d")
excel_output = f"{TICKER}_{NOW}.xlsx"
image_output = f"{TICKER}.png"
html_output = f"{TICKER}.html"
```

### Setup Naas scheduler


```python
naas.scheduler.add(cron="0 9 * * *")

# if you want to delete the scheduler, uncoment the line below and execute the cell
# naas.scheduler.delete() 
```

## Model

### Get dataset from Yahoo Finance


```python
df_yahoo = yahoofinance.get(tickers=TICKER,
                            date_from=date_from,
                            date_to=date_to).dropna().reset_index(drop=True)

# Display dataframe
df_yahoo.tail(5)
```

### Add prediction columns


```python
df_predict = prediction.get(dataset=df_yahoo,
                            date_column='Date',
                            column="Close",
                            data_points=DATA_POINT,
                            prediction_type="all").sort_values("Date", ascending=False).reset_index(drop=True)
# Display dataframe
df_predict.head(int(DATA_POINT)+5)
```

### Plot linechart


```python
fig = plotly.linechart(df_predict,
                       x="Date",
                       y=["Close", "ARIMA", "SVR", "LINEAR", "COMPOUND"],
                       showlegend=True,
                       title=f"{TICKER} predictions as of today, for next {str(DATA_POINT)} days.")
```

### Set actual data and variation


```python
def get_variation(df):
    df = df.sort_values("Date", ascending=False).reset_index(drop=True)
    
    # Get value and value comp
    datanow = df.loc[0, "Close"]
    datayesterday = df.loc[1, "Close"]
    
    # Calc variation en value and %
    varv = datanow - datayesterday
    varp = (varv / datanow)
    return datanow, datayesterday, varv, varp

DATANOW, DATAYESTERDAY, VARV, VARP = get_variation(df_yahoo)
print("Value today:", DATANOW)
print("Value yesterday:", DATAYESTERDAY)
print("Var. in value:", VARV)
print("Var. in %:", VARP)
```

### Save and share your data in Excel


```python
df_predict.to_excel(excel_output)

# Share output with naas
link_excel = naas.asset.add(excel_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(excel_output)
```

### Save and share your graph in PNG


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
link_image = naas.asset.add(image_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```

### Save and share your graph in HTML


```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
link_html = naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```

## Output

### Create/Update page in Notion DB


```python
def update_notion_db(database_url, title, value=0, varv=0, varp=0, image_link=None, html_link=None, excel_link=None):
    # Decode database id
    database_id = database_url.split("/")[-1].split("?v=")[0]
    
    # Get pages from notion database
    pages = notion.connect(NOTION_TOKEN).database.query(database_id, query={})
    
    # Create or update page
    page_new = True
    for page in pages:
        page_temp = page.df()
        page_id = page_temp.loc[page_temp.Name == "Name", "Value"].values[0]
        if page_id == title:
            page_new = False
            break
    try:
        if page_new:
            page = notion.connect(NOTION_TOKEN).Page.new(database_id=database_id).create()
            page.title("Name", title)
            
        # Check if image already exists
        blocks = page.get_blocks()
        for block in blocks:
            content_block = getattr(block, block.type)
            if block.type == "image":
                image_url = block.image.external.url
                if image_url == image_link:
                    image_link = None
            if block.type == "paragraph":
                if len(block.paragraph.text) > 0:
                    text = block.paragraph.text[0].text.content
                    if text == "Open dynamic graph":
                        html_link = None
                    if text == "Download Excel":
                        excel_link = None
        if image_link:
            page.image(image_link)
        if html_link:
            res = page.paragraph("Open dynamic graph")
            res.paragraph.text[0].href = html_link
            res.paragraph.text[0].text.link = Link(html_link)
        if excel_link:
            res = page.paragraph("Download Excel")
            res.paragraph.text[0].href = excel_link
            res.paragraph.text[0].text.link = Link(excel_link)
                                    
        # Update dynamic properties
        page.select("Status", "OK")
        page.number("Value", round(float(value), 0))
        page.number("Var (value)", round(float(varv), 0))
        page.number("Var (%)", round(float(varp), 4))
        page.date("Updated at", datetime.now(pytz.timezone("Europe/Paris")).strftime("%Y-%m-%d %H:%M:%S%z"))

        # Create page in Notion
        page.update()
        print(f"✅ Page '{title}' updated in Notion.")
    except Exception as e:
        print(f"❌ Error updating {title}")
        return e
        
update_notion_db(DATABASE_URL, PAGE_TITLE, DATANOW, VARV, VARP, link_image, link_html, link_excel)
```


```python

```
