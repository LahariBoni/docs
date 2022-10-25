<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Sheets/Google_Sheets_Send_data_to_MongoDB.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Sheets+-+Send+data+to+MongoDB:+Error+short+description">🚨 Bug report</a>

**Tags:** #googlesheets #mongodb #nosql #operations #automation

**Author:** [Oketunji Oludolapo](https://www.linkedin.com/in/oludolapo-oketunji/)

This notebook will help you send data from your spreadsheet to your MongoDB database collection

**How To Use Template:** [Video](https://drive.google.com/file/d/1fSeOXVjVj0gXE-hr1oSCVGyoV2HISczp/view?usp=sharing)

## Input


```python
from naas_drivers import mongo, gsheet
import pandas as pd
import naas
```

### Setup Google Sheet
- Share your Google Sheet with our service account : 🔗 naas-share@naas-gsheets.iam.gserviceaccount.com


```python
spreadsheet_id = "------"
sheet_name = "Sheet1"
```

### Setup MongoDB
- Get your user, password and connection URL details from your MongoDB Atlas Cluster
- **How To get Required MongoDB details**: [Article](https://towardsdev.com/part-6-7-of-python-mongodb-tutorial-series-for-data-scientists-connecting-mongodb-atlas-to-python-d7169445fca1)


```python
user = "your user"
passwd = "your password"
host = "Your Connection URL"
port = 9090

collection_name = "COLLECTION NAME"
db_name = "DATABASE NAME"
```

### Setup Naas


```python
naas.scheduler.add(cron='0 9 * * *') #Send in production this notebook and run it, every day at 9:00.

# use this to delete your automation
#naas.scheduler.delete()
```

## Model

### Get data from Google Sheets


```python
df = gsheet.connect(spreadsheet_id).get(sheet_name=sheet_name)
```

## Output

### Send data to MongoDB


```python
mongo.connect(host, port, user, passwd).send(df,
                                             collection_name,
                                             db_name,
                                             replace=True)
```
