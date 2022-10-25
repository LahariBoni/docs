<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Sheets/Google_Sheets_Get_data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Sheets+-+Get+data:+Error+short+description">🚨 Bug report</a>

**Tags:** #googlesheets #gsheet #data #naas_drivers #operations #snippet #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook fetch data from a Google Sheets spreadsheet.

## Input

### Import library


```python
from naas_drivers import gsheet
```

### Setup Google Sheets
- Share your Google Sheets spreadsheet with our service account : 🔗 naas-share@naas-gsheets.iam.gserviceaccount.com


```python
SPREADSHEET_URL = "https://docs.google.com/spreadsheets/d/1RdwdYXDFDSFSFxxxxxxxx/edit#gid=XXXXXXXX33"
SHEET_NAME = "MY_SHEET"
```

## Model

### Get data from Google Sheets spreadsheet


```python
df = gsheet.connect(SPREADSHEET_URL).get(sheet_name=SHEET_NAME)
```

## Output

### Display result


```python
df
```
