<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YahooFinance/YahooFinance_Get_Stock_Update.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YahooFinance+-+Get+Stock+Update:+Error+short+description">🚨 Bug report</a>

**Tags:** #yahoofinance #usdinr #plotly #investors #analytics #automation

**Author:** [Megha Gupta](https://github.com/megha2907)

 Description: With this template you will get INR USD rate visualized on a chart

## Input

### Import Libraries


```python
import naas 
from naas_drivers import yahoofinance, plotly
import markdown2
from IPython.display import Markdown as md
```

### Setup Yahoo parameters

👉 Here you can input:<br>
- yahoo ticker : get tickers <a href='https://finance.yahoo.com/trending-tickers?.tsrc=fin-srch'>here</a>
- date from
- date to


```python
TICKER = 'INR=X'
date_from = -30
date_to = 'today'
```

### Setup your email parameters
👉 Here you can input your sender email and destination email

Note: emails are sent from notification@naass.ai by default


```python
email_to = ["template@naas.ai"]
email_from = None
```

## Model

### Get the data from yahoo finance using naas drivers


```python
#data cleaning
df = yahoofinance.get(TICKER, date_from=date_from, date_to = date_to)
df = df.dropna()# drop the na values from the dataframe
df.reset_index(drop=True)
df = df.sort_values("Date", ascending=False).reset_index(drop=True)
df.head()
```

### Extract value from data


```python
LASTOPEN = round(df.loc[0, "Open"], 2)
LASTCLOSE = round(df.loc[0, "Close"], 2)
YESTERDAYOPEN = round(df.loc[1, "Open"], 2)
YESTERDAYCLOSE = round(df.loc[1, "Close"], 2)
MAXRATE = round(df['Open'].max(),2)
MXDATEOPEN = df.loc[df['Open'].idxmax(), "Date"].strftime("%Y-%m-%d")
MINRATE = round(df['Open'].min(),2)
MNDATEOPEN = df.loc[df['Open'].idxmin(), "Date"].strftime("%Y-%m-%d")
```

### Plot the data


```python
last_date = df.loc[df.index[0], "Date"].strftime("%Y-%m-%d")

output = plotly.linechart(df,
                          x="Date",
                          y=['Open','Close'],
                          title=f"<b>INR USD rates of last month</b><br><span style='font-size: 13px;'>Last value as of {last_date}: Open={LASTOPEN}, Close={LASTCLOSE}</span>")
```

## Output

### Save the dataset in csv


```python
df.to_csv(f"{TICKER}_LastMonth.csv", index=False)
```

### Create markdown template


```python
%%writefile message.md
Hello world,

The **TICKER** price is Open LASTOPEN and Close LASTCLOSE right now. <br>
**Yesterday Open**: YESTERDAYOPEN <br>
**Yesterday Close**: YESTERDAYCLOSE <br>    
The Max Open rate of **TICKER** was on MXDATEOPEN which was MAXRATE. <br>
The Min Open rate of **TICKER** was on MNDATEOPEN which was MINRATE. <br>

Attached is the excel file for your reference. <br>

Have a nice day.
<br>

PS: You can [send the email again](link_webhook) if you need a fresh update.<br>
<div><strong>Full Name</strong></div>
<div>Open source lover | <a href="http://www.naas.ai/" target="_blank">Naas</a></div>
<div>+ 33 1 23 45 67 89</div>
<div><small>This is an automated email from my Naas account</small></div>
```

### Add email template as dependency


```python
naas.dependency.add("message.md")
```

### Replace values in template


```python
markdown_file = "message.md"
content = open(markdown_file, "r").read()
md = markdown2.markdown(content)
md
```


```python
post = md.replace("LASTOPEN", str(LASTOPEN))
post = post.replace("LASTCLOSE", str(LASTCLOSE))
post = post.replace("YESTERDAYOPEN", str(YESTERDAYOPEN))
post = post.replace("YESTERDAYCLOSE", str(YESTERDAYCLOSE))
post = post.replace("MXDATEOPEN", str(MXDATEOPEN))
post = post.replace("MAXRATE", str(MAXRATE))
post = post.replace("MNDATEOPEN", str(MNDATEOPEN))
post = post.replace("MINRATE", str(MINRATE))
post = post.replace("TICKER", str(TICKER))
post
```

### Add webhook to run your notebook again


```python
link_webhook = naas.webhook.add()
```

### Send by email


```python
subject = f"📈 {TICKER} Open and close rates as of today"
content = post
files = [f"{TICKER}_LastMonth.csv"]

naas.notification.send(email_to=email_to,
                       subject=subject,
                       html=content,
                       email_from=email_from,
                       files=files)
```

### Schedule your notebook
Please uncomment and run the cell below to schedule your notebook everyday at 8:00 during business days


```python
# import naas
# naas.scheduler.add("0 8 1-5 * *")
```
