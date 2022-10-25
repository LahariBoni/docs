<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Generate_Google_Sheets_rows_for_new_items_in_Notion_database.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Generate+Google+Sheets+rows+for+new+items+in++database:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #operations #automation #googlesheets

**Author:** [Pooja Srivastava](https://www.linkedin.com/in/pooja-srivastava-in/)

Here, you will create new row in your Google Sheets spreadsheet for new items your Notion database.

## Input

### Import librairies


```python
from naas_drivers import notion, gsheet
import pandas as pd
```

### Setup Notion
- [Get your Notion integration token](https://docs.naas.ai/drivers/notion)
- Share integration with your database


```python
# Enter Token API
NOTION_TOKEN = "*****"

# Enter Database URL
DATABASE_URL = "https://www.notion.so/********"

#Unique column name for notion
col_unique_notion = 'Name'
```

### Setup Google Sheets
- Share your sheet with our service account : 🔗 naas-share@naas-gsheets.iam.gserviceaccount.com


```python
# Spreadsheet URL
SPREADSHEET_URL = "------"

# Sheet name
SHEET_NAME = "Sheet1"

#Unique column# for gsheets
col_unique_gsheet = 'Name'
```

### Setup Naas


```python
# Schedule your notebook every hours
naas.scheduler.add(cron="0 * * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get dataframe from Notion database


```python
db_notion = notion.connect(NOTION_TOKEN).database.get(DATABASE_URL)
df_notion = db_notion.df()
```

### Get dataframe Google Sheets spreasheet


```python
df_gsheet = gsheet.connect(SPREADSHEET_URL).get(sheet_name=SHEET_NAME)
df_gsheet
```

### Compare Notion database with Google Sheets


```python
#Iterate through all rows in Notion database and find match in Google Sheets
#If no match is found then add data to df_difference dataframe

df_difference = pd.DataFrame()
for index,row in df_notion.iterrows():
    x = row[col_unique_notion]
    if not (x == df_gsheet[col_unique_gsheet]).any():
        df_difference = df_difference.append(df_notion.loc[index])
```

## Output

### Add new rows in Google Sheets


```python
# Send data to Google Sheets
gsheet.connect(SPREADSHEET_URL).send(
    sheet_name=SHEET_NAME,
    data=df_difference,
    append=True,
)
```
