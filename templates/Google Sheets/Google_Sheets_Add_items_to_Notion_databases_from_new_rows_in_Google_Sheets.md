<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Sheets/Google_Sheets_Add_items_to_Notion_databases_from_new_rows_in_Google_Sheets.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Sheets+-+Add+items+to+Notion+databases+from+new+rows+in:+Error+short+description">🚨 Bug report</a>

**Tags:** #googlesheets #productivity #operations #automation #notion

**Author:** [Pooja Srivastava](https://www.linkedin.com/in/pooja-srivastava-bb037649/)

This notebook does the following tasks: 
- Schedule the notebook to run every 15 minutes
- Connect with Gsheet and get all rows using the gsheet driver's get method
- Connect with NotionDB and get all pages using the Notion Databases driver's get method
- Compare the list of pages from notion db with the rows returned from gsheet and add non matching rows to a list
- Create new pages in Notion db for each row in the non matching list using the Notion Pages driver's create method
- For each created page, add the properties and values from gsheet and update in notion using the Notion Pages driver's update methods

Pre-requisite: 
1. Gsheets : Please share your Google Sheet with our service account
   For the driver to fetch the contents of your google sheet, you need to share it with the service account linked with Naas. 🔗 naas-share@naas-gsheets.iam.gserviceaccount.com 
2. Notion : Create a notion integration and use the secret key to share the database. <a href = 'https://docs.naas.ai/Notion-7435020d01a549a9a0060c47ea808fd4'> Refer the documentation here</a>

## Input

### Import libraries


```python
import pandas as pd
import naas
from naas_drivers import notion, gsheet
```

### Setup Google Sheets


```python
# Enter spreadsheet_id and sheet name
spreadsheet_id = "****"
sheet_name = "YOUR_SHEET"

#Unique column# for gsheets
col_unique_gsheet = 'Name'
```

### Setup Notion


```python
# Enter Notion Token API and Database URL
notion_token = "****"
notion_database_url = "https://www.notion.so/YOURDB"

#Unique column name for notion
col_unique_notion = 'Name'
```

### Setup Naas


```python
#Schedule the notebook to run every 15 minutes
naas.scheduler.add(cron="*/15 * * * *")

# To delete your scheduler, uncomment the line below and execute the cell 
# naas.scheduler.delete()
```

## Model

### Get Gsheet and Notion data


```python
#Connect with Gsheet and get all data in a dataframe
gsheet.connect(spreadsheet_id)
df_gsheet = gsheet.get(sheet_name=sheet_name)
```


```python
#Connect with Notion db and get all pages in a dataframe
database = notion.connect(notion_token).database.get(notion_database_url)
df_notion = database.df()
```

### Compare Gsheet and Notion data


```python
#Iterate through all rows in Gsheet and find match in Notion db
#If no match is found then add data to df_difference dataframe

df_difference = pd.DataFrame()
for index,row in df_gsheet.iterrows():
    x = row[col_unique_gsheet]
    if not (x == df_notion[col_unique_notion]).any():
        df_difference = df_difference.append(df_gsheet.loc[index])
```

## Output

### Create a new page in Notion if it does not match with Gsheet


```python
#Create a new page in notion db for each row in df_difference dataframe
if(df_difference.empty == False):    
    for index, row in df_difference.iterrows():
        page = notion.connect(notion_token).page.create(database_id=notion_database_url, title=row[col_unique_gsheet])
        #Add all properties here and map with respective column in row
        page.select('Type', row['Type'])
        page.select('Status', row['Status'])
        page.rich_text('Summary', row['Summary'])
        page.update()
    print("The gsheets rows synced successfuly to Notion DB")
else:     
    print("No new rows in Gsheet to sync to Notion DB")
```
