<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Google%20Analytics/Google_Analytics_Send_visitors_traffic_graph_and_trends_prediction_to_Slack_channel.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Google+Analytics+-+Send+visitors+traffic+graph+and+trends+prediction+to+Slack+channel:+Error+short+description">🚨 Bug report</a>

**Tags:** #googleanalytics #analytics #marketing #dataframe

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maxime-jublou)

This notebook is aimed to help you to get insight about your website traffic right into your Slack Workspace.

## Input

### Import libraries


```python
# Used for scheduling and secrets.
import naas

# Used to interact with Google Analytics.
from naas_drivers import googleanalytics

# Used to predict future traffic.
from naas_drivers import prediction

# Used to plot.
from naas_drivers import plotly

# Used to send Slack notification.
from naas_drivers import slack

# Used to generate a plot.
import plotly.graph_objects as go

# Used to apply logic on data.
import pandas as pd

# Used to handle dates.
from datetime import datetime

# Used to convert Slack Blocks.
import json
```

### Setup Google Analytics

👉 Create your own <a href="">Google API JSON credential</a>


```python
# Get your credential from Google Cloud Platform
json_path = 'naas-googleanalytics.json'

# Get view id from google analytics
view_id = "228952707"

# Setup your data parameters
dimensions = "daily" #hourly, daily, weekly, monthly
start_date = "30daysAgo" #XdaysAgo or date in ISO format %Y-%m-%d
end_date = "today" #Today or date in ISO format %Y-%m-%d
```

### Setup Slack


```python
SLACK_TOKEN = 'xoxb-111111111111-111111111111-abcdef123'
SLACK_CHANNEL = '#operation-notifications'
```

### Setup Prediction
👉 Here you can change the number of data points you want the prediction will be performed on


```python
DATA_POINT = 20
```

### Setup Outputs


```python
# Chart title
title = "Visitors"

# Outputs path
NOW = datetime.now().strftime("%Y-%m-%d")
excel_output = f"{view_id}_{NOW}.xlsx"
image_output = f"{view_id}.png"
html_output = f"{view_id}.html"
```

### Setup Naas scheduler


```python
# Adding Google Analytics credentials as a dependency file.
naas.dependency.add(path=json_path)

# Let's run this everyday at 9 am 🚀
naas.scheduler.add(cron="0 9 * * *")

# this notebook will run each week until de-scheduled
# to de-schedule this notebook, simply run the following command: 
# naas.scheduler.delete()
```

## Model

### Get trend


```python
df = googleanalytics.connect(json_path, view_id).users.get_trend(dimensions, start_date, end_date)
df
```

### Add prediction columns


```python
df_predict = prediction.get(dataset=df,
                            date_column='DATE',
                            column="VALUE",
                            data_points=DATA_POINT,
                            prediction_type="all").sort_values("DATE", ascending=False).reset_index(drop=True)
# Display dataframe
df_predict.head(int(DATA_POINT)+5)
```

### Plot linechart


```python
fig = plotly.linechart(df_predict,
                       x="DATE",
                       y=["VALUE", "ARIMA", "SVR", "LINEAR", "COMPOUND"],
                       showlegend=True,
                       title=f"predictions as of today, for next {str(DATA_POINT)} days.")
```


```python
def get_variation(df):
    df = df.sort_values("DATE", ascending=False).reset_index(drop=True)
    
    # Get value and value comp
    datanow = df.loc[0, "VALUE"]
    datayesterday = df.loc[1, "VALUE"]
    
    # Calc variation en value and %
    varv = datanow - datayesterday
    varp = (varv / datanow)
    
    # Format result
    datanow = "{:,.2f}".format(round(datanow, 1))
    datayesterday = "{:,.2f}".format(round(datayesterday, 1))
    varv = "{:+,.2f}".format(varv)
    varp = "{:+,.2%}".format(varp)
    return datanow, datayesterday, varv, varp

DATANOW, DATAYESTERDAY, VARV, VARP = get_variation(df)
variations = f"Value today: {DATANOW}\nValue yesterday: {DATAYESTERDAY}\nVar. in value: {VARV}\nVar. in %: {VARP}"
print(variations)
```

### Set predict data


```python
def get_prediction(df, prediction):
    data = df.loc[0, prediction]
    
    # Format result
    data = "{:,.2f}".format(round(data, 1))
    return data

predictions = f'Value ARIMA: {get_prediction(df_predict, "ARIMA")}\nValue SVR: {get_prediction(df_predict, "SVR")}\nValue LINEAR: {get_prediction(df_predict, "LINEAR")}\nValue COMPOUND: {get_prediction(df_predict, "COMPOUND")}'
print(predictions)
```

## Output

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

### Create webhook to run your notebook again


```python
link_webhook = naas.webhook.add()
```

### Prepare Slack message


```python
slack_blocks = [
		{
			"type": "section",
			"text": {
				"type": "mrkdwn",
				"text": "👋 Hello your daily informations about your site traffic is freshly baked!\n\n"
			}
		},
		{
			"type": "section",
			"text": {
				"type": "mrkdwn",
				"text": variations
			}
		},
		{
			"type": "section",
			"text": {
				"type": "mrkdwn",
				"text": predictions
			}
		},
		{
			"type": "image",
			"title": {
				"type": "plain_text",
				"text": "Website traffic statistics and prediction.",
				"emoji": True
			},
			"image_url": link_image,
			"alt_text": "Predictions"
		},
		{
			"type": "section",
			"text": {
				"type": "mrkdwn",
				"text": "Open interactive graph in full screen"
			},
			"accessory": {
				"type": "button",
				"text": {
					"type": "plain_text",
					"text": "Open 👆",
					"emoji": True
				},
				"value": "Open interactive graph",
				"url": link_html,
				"action_id": "button-action"
			}
		}
	]
```


```python
slack.connect(SLACK_TOKEN).send(SLACK_CHANNEL, '', blocks=json.dumps(slack_blocks, ensure_ascii=False))
```
