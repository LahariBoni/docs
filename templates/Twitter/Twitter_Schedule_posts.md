<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Twitter/Twitter_Schedule_posts.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Twitter+-+Schedule+posts:+Error+short+description">🚨 Bug report</a>

**Tags:** #twitter #automation #ifttt #naas_drivers #gsheet #content

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import library


```python
from datetime import datetime
import naas_drivers
import naas
```

### Variables


```python
spreadsheet_id = "1rFzw8eeVNXyD5CEUjnpxVn_iA2hzabYKU6pRdmnZ3FQ"
```

## Model

### Get data from Gsheet


```python
df = naas_drivers.gsheet.connect(spreadsheet_id).get(
    sheet_name="Sheet1"
)
df
```

### Get current date and time details


```python
now = datetime.now().replace(second=0, microsecond=0) #the Naas scheduler only allow minutes
now
```


```python
# dd/mm/YY H:M:S
dt_string = now.strftime("%d/%m/%Y %H:%M:%S") # To mach date time format in google sheet
dt_string
```

### Schedule check every minute
If BROADCAST_DATE matches the current time then send twit from text column.


```python
#naas.scheduler.delete()
naas.scheduler.add(recurrence="* * * * *") # to run every minute
```

## Output

### If current date and column date in Gsheet matches then post


```python
for i in range(len(df)):
    print(dt_string, df['BROADCAST_DATE'][i])
    if dt_string == df['BROADCAST_DATE'][i]:
        twitter_post = df['TEXT'][i]
        event = "naas_demo"
        key = "ke4AigvXI5-EABaowdLt4fju1aOUxeMxSXQoN8FVyA"
        data = { "value1": twitter_post }
        result = naas_drivers.ifttt.connect(key).send(event, data)   
```
