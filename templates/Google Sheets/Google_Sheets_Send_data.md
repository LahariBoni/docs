<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Sheets/Google_Sheets_Send_data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Sheets+-+Send+data:+Error+short+description">🚨 Bug report</a>

**Tags:** #googlesheets #gsheet #data #naas_drivers #operations #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook send data to a Google Sheets spreadsheet.

## Input

### Import library


```python
from naas_drivers import gsheet
import pandas as pd
```

### Setup Google Sheets
- Share your Google Sheets spreadsheet with our service account : 🔗 naas-share@naas-gsheets.iam.gserviceaccount.com


```python
SPREADSHEET_URL = "https://docs.google.com/spreadsheets/d/1RdwdYXDFDSFSFxxxxxxxx/edit#gid=XXXXXXXX33"
SHEET_NAME = "MY_SHEET"
```

### Variables


```python
data = [{ "name": "Jean", "email": "jean@appleseed.com" },
        { "name": "Bunny", "email": "bunny@appleseed.com" }]
df = pd.DataFrame(data)
```

## Model

### Send dataframe to Google Sheets spreadsheet


```python
result = gsheet.connect(SPREADSHEET_URL).send(sheet_name=SHEET_NAME,
                                              data=df,
                                              append=False)
```

## Output

### Display result


```python
result
```
