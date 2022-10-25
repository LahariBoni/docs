<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Slack/Slack_Follow_number_of_users_in_workspace.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Slack+-+Follow+number+of+users+in+workspace:+Error+short+description">🚨 Bug report</a>

**Tags:** #slack #plotly #html #image #csv #marketing #automation #analytics


**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)

With this notebook, you can follow your number of Slack users in workspace over the time
<br/>References :
- Defining required scopes for slack app:
    - https://api.slack.com/scopes/channels:read
    - https://api.slack.com/scopes/groups:read
    - https://api.slack.com/scopes/im:read
    - https://api.slack.com/scopes/mpim:read
    - https://api.slack.com/scopes/users:read
- Slack SDK to use : [https://github.com/slackapi/python-slack-sdk](https://github.com/slackapi/python-slack-sdk)


## Input


### Import libraries



```python
!pip install slack-sdk --user
```


```python
import naas
import pandas as pd
import plotly.graph_objects as go
from slack_sdk import WebClient
from datetime import datetime
```

### Setup Slack



```python
SLACK_BOT_TOKEN = "xoxb-232887839156-1673274923699-vTF6xxxxxxxxxx"
```


```python
client = WebClient(token=SLACK_BOT_TOKEN)
```

### Get channel ID's and use the one required


```python
def get_channel_ids():
    channel_id = {}
    for channel_info in client.conversations_list().data['channels']:
        key, value = channel_info['name'], channel_info['id']
        channel_id[key] = value
    return channel_id

channel_dict = get_channel_ids()
channel_dict
```

### Setup Outputs



```python
# Outputs
title = "Title of your chart"
name_output = "My_output"
csv_output = f"{name_output}.csv"
html_output = f"{name_output}.html"
image_output = f"{name_output}.png"
```

### Setup Naas



```python
# Schedule your notebook every hour
naas.scheduler.add(cron="0 * * * *")

#-> Uncomment the line below and execute this cell to delete your scheduler
# naas.scheduler.delete()
```

## Model


### List users from Slack workspace



```python
def list_users():
    df = pd.DataFrame()
    idx=0
    for user_data in client.users_list().data['members']:
        if ('real_name' in user_data and user_data['real_name'] != 'Slackbot') and not user_data['is_bot']:
            df.loc[idx,'NAME'] = user_data['profile']['real_name']
            df.loc[idx,'ID'] = user_data['id']
            df.loc[idx,'FIRST_VIEWED_AT'] = datetime.fromtimestamp(user_data['updated'])
            idx+=1
    
    return df

df_slack = list_users()
df_slack
```

### Get historical data



```python
def list_users_histo():
    # Load csv file containing already seen users
    try:
        df = pd.read_csv(csv_output)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()

    return df

df_slack_histo = list_users_histo()
df_slack_histo 
```

### Append new users to historical data



```python
# Append new users to historical data with today's date.

def merge_dataframes(df_slack, df_slack_histo):
	# Add new users + date. It could be a two columns dataframe ['EMAIL', 'DATE_EXTRACT']
	
    if len(df_slack_histo) == 0:
        return df_slack
    else:
        historical_data = df_slack_histo.ID.to_list()
        for idx, row in df_slack.iterrows():
            if row['ID'] not in historical_data:
                df_slack_histo = df_slack_histo.append(row)
        
        return df_slack_histo

merged_df = merge_dataframes(df_slack, df_slack_histo)
merged_df
```

### Get trend



```python
def get_trend(df,
              date_col_name='FIRST_VIEWED_AT',
              value_col_name="ID",
              date_order='asc'):
    
    # Format date
    df[date_col_name] = pd.to_datetime(df[date_col_name]).dt.strftime("%Y-%m-%d")
    df = df.groupby(date_col_name, as_index=False).agg({value_col_name: "count"})
    d = datetime.now().date()
    d2 = df.loc[df.index[0], date_col_name]
    idx = pd.date_range(d2, d, freq = "D")
    
    df.set_index(date_col_name, drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df[date_col_name] = pd.DatetimeIndex(df.index)
    # Calc sum cum
    df["value_cum"] = df.agg({value_col_name: "cumsum"})
    return df.reset_index(drop=True)

df_trend = get_trend(merged_df)
df_trend
```

### Create linechart



```python
def create_linechart(df, label, value, title):
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Scatter(
            x=df[label],
            y=df[value],
            mode="lines",
        )
    )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        title=title,
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_trend, label="FIRST_VIEWED_AT", value="value_cum", title=title)
```

## Output


### Save and share your csv file



```python
# Save your dataframe in CSV
merged_df.to_csv(csv_output, index=False)

# Share output with naas
naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(csv_output)
```

### Save and share your graph in HTML



```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```

### Save and share your graph in image



```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
naas.asset.add(image_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
