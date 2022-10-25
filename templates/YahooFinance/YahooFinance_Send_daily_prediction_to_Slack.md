<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YahooFinance/YahooFinance_Send_daily_prediction_to_Slack.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YahooFinance+-+Send+daily+prediction+to+Slack:+Error+short+description">🚨 Bug report</a>

**Tags:** #yahoofinance #trading #markdown #prediction #plotly #slack #naas_drivers #scheduler #naas #investors #automation #analytics

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

With this template, you can get any ticker available in [Yahoo finance](https://finance.yahoo.com/quote/TSLA/),add predictions and send a custom message on Slack.<br> 

## Input

### Import libraries


```python
import naas
from naas_drivers import prediction, yahoofinance, plotly, slack
import markdown2
from datetime import datetime
import naas
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

### Setup Slack
- [Create new app](https://api.slack.com/apps) in Slack
- Go to "OAuth & Permissions", "Bot Token Scopes" and add scope : **chat:write.public**
- Go to "Install App" section and click on button "Install to workspace"
- Copy/Paste your token in cell below


```python
SLACK_TOKEN = "xoxb-XXXXXXX"
SLACK_CHANNEL = "demo-naas"

# Markdown template created on your local env
SLACK_CONTENT_MD = "slack_content.md"
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
    
    # Format result
    datanow = "${:,.2f}".format(round(datanow, 1))
    datayesterday = "${:,.2f}".format(round(datayesterday, 1))
    varv = "{:+,.2f}".format(varv)
    varp = "{:+,.2%}".format(varp)
    return datanow, datayesterday, varv, varp

DATANOW, DATAYESTERDAY, VARV, VARP = get_variation(df_yahoo)
print("Value today:", DATANOW)
print("Value yesterday:", DATAYESTERDAY)
print("Var. in value:", VARV)
print("Var. in %:", VARP)
```

### Set predict data


```python
def get_prediction(df, prediction):
    data = df.loc[0, prediction]
    
    # Format result
    data = "${:,.2f}".format(round(data, 1))
    return data

ARIMA = get_prediction(df_predict, "ARIMA")
print("Value ARIMA:", ARIMA)
SVR = get_prediction(df_predict, "SVR")
print("Value SVR:", SVR)
LINEAR = get_prediction(df_predict, "LINEAR")
print("Value LINEAR:", LINEAR)
COMPOUND = get_prediction(df_predict, "COMPOUND")
print("Value COMPOUND:", COMPOUND)
```

### Save data in Excel


```python
df_predict.to_excel(excel_output)
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

### Save and share your graph in PNG


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
link_image = naas.asset.add(image_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```

### Create Slack template


```python
%%writefile $SLACK_CONTENT_MD
Hey <!here>

The *TICKER* price is *DATANOW* right now, VARV vs yesterday (VARP).
Yesterday close : DATAYESTERDAY

In +DATA_POINT days, basic ML models predict the following prices: 

- *arima*: ARIMA
- *svr*: SVR
- *linear*: LINEAR
- *compound*: COMPOUND

<link_html|Open dynamic chart>
```

### Replace values in templates


```python
def replace_value(md):
    post = md.replace("DATANOW", str(DATANOW))
    post = post.replace("TICKER", str(TICKER))
    post = post.replace("DATAYESTERDAY", str(DATAYESTERDAY))
    post = post.replace("VARV", str(VARV))
    post = post.replace("VARP", str(VARP))
    post = post.replace("LINEAR", str(LINEAR))
    post = post.replace("SVR", str(SVR))
    post = post.replace("COMPOUND", str(COMPOUND))
    post = post.replace("ARIMA", str(ARIMA))
    post = post.replace("DATA_POINT", str(DATA_POINT))
    post = post.replace("link_image", str(link_image))
    post = post.replace("link_html", str(link_html))
    return post
```

### Create  Slack message


```python
content = open(SLACK_CONTENT_MD, "r").read()
slack_message = replace_value(content)
slack_message
```

## Output

### Send Slack message


```python
slack.connect(SLACK_TOKEN).send(
    SLACK_CHANNEL,
    slack_message,
)
```
