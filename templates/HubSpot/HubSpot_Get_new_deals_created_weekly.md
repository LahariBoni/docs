<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Get_new_deals_created_weekly.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Get+new+deals+created+weekly:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #deal #scheduler #asset #html #png #csv #naas_drivers #naas #analytics #automation #image #plotly #notification #email

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
from naas_drivers import hubspot, emailbuilder
from datetime import datetime, timedelta
import pandas as pd
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import naas
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_KEY = "YOUR_API_KEY"
```

### Select your pipeline ID
Here below you can select your pipeline.<br>
If not, all deals will be taken for the analysis


```python
df_pipelines = hubspot.connect(HS_API_KEY).pipelines.get_all()
df_pipelines
```


```python
pipeline_id = "000000"
```

### Setup Naas


```python
# Scheduler at 08:00 on Monday and Friday.
naas.scheduler.add(cron="0 8 * * 1,5")

#-> Uncomment the line below (by removing the hashtag) to remove your scheduler
# naas.scheduler.delete()
```


```python
# Email info
EMAIL_TO = "YOUR_EMAIL"
EMAIL_SUBJECT = "[HubSpot] Weekly update : New deals created"
```

### Setup Outputs


```python
name_output = "HubSpot_new_deals_weekly"
csv_output = f"{name_output}.csv"
image_output = f"{name_output}.png"
html_output = f"{name_output}.html"
```

## Model

### Get all deals


```python
df_deals = hubspot.connect(HS_API_KEY).deals.get_all()
df_deals
```

### Create trend data


```python
def get_trend(df_deals, pipeline):
    df = df_deals.copy()
    # Filter data
    df = df[df["pipeline"].astype(str) == str(pipeline)]
    
    # Prep data
    df["createdate"] = pd.to_datetime(df["createdate"])
    df["amount"] = df.apply(lambda row: float(row["amount"]) if str(row["amount"]) not in ["None", ""] else 0, axis=1)
    
    # Calc by week
    df = df.groupby(pd.Grouper(freq='W', key='createdate')).agg({"hs_object_id": "count", "amount": "sum"}).reset_index()
    df["createdate"] = df["createdate"] + timedelta(days=-1)
    df = pd.melt(df, id_vars="createdate")
    
    # Rename col
    to_rename = {
        "createdate": "LABEL_ORDER",
        "variable": "GROUP",
        "value": "VALUE"
    }
    df = df.rename(columns=to_rename).replace("hs_object_id", "No of deals").replace("amount", "Amount")
    df["YEAR"] = df["LABEL_ORDER"].dt.strftime("%Y")
    df = df[df["YEAR"] == datetime.now().strftime("%Y")]
    df["LABEL"] = df["LABEL_ORDER"].dt.strftime("%Y-W%U")
    df["LABEL_ORDER"] = df["LABEL_ORDER"].dt.strftime("%Y%m%d")
    
    # Calc variation
    df_var = pd.DataFrame()
    groups = df.GROUP.unique()
    for group in groups:
        tmp = df[df.GROUP == group].reset_index(drop=True)
        for idx, row in tmp.iterrows():
            if idx == 0:
                value_n1 = 0
            else:
                value_n1 = tmp.loc[tmp.index[idx-1], "VALUE"]
            tmp.loc[tmp.index[idx], "VALUE_COMP"] = value_n1
        df_var = pd.concat([df_var, tmp]).fillna(0).reset_index(drop=True)
    df_var["VARV"] = df_var["VALUE"] - df_var["VALUE_COMP"]
    df_var["VARP"] = df_var["VARV"] / abs(df_var["VALUE_COMP"])
    
    # Prep data
    df_var["VALUE_D"] = df_var["VALUE"].map("{:,.0f}".format).str.replace(",", " ")
    df_var["VARV_D"] = df_var["VARV"].map("{:,.0f}".format).str.replace(",", " ")
    df_var.loc[df_var["VARV"] > 0, "VARV_D"] = "+" + df_var["VARV_D"]
    df_var["VARP_D"] = df_var["VARP"].map("{:,.0%}".format).str.replace(",", " ")
    df_var.loc[df_var["VARP"] > 0, "VARP_D"] = "+" + df_var["VARP_D"]
    
    # Create hovertext
    df_var["TEXT"] = ("<b>Deal created as of " + df_var["LABEL"] + " : </b>" + 
                      df_var["VALUE_D"] + "<br>" + 
                      df_var["VARP_D"] + " vs last week (" + df_var["VARV_D"] + ")")
    return df_var

df_trend = get_trend(df_deals, pipeline_id)
df_trend
```

### Plotting a barchart


```python
def create_barchart(df, label, group, value, varv, varp):    
    # Create figure with secondary y-axis
    fig = make_subplots(specs=[[{"secondary_y": True}]])

    # Add traces
    df1 = df[df[group] == "No of deals"].reset_index(drop=True)[:]
    total_volume = "{:,.0f}".format(df1[value].sum()).replace(",", " ")
    var_volume = df1.loc[df1.index[-1], varv]
    positive = False
    if var_volume > 0:
        positive = True
    var_volume = "{:,.0f}".format(var_volume).replace(",", " ")
    if positive:
        var_volume = f"+{var_volume}"
    fig.add_trace(
        go.Bar(
            name="No of deals",
            x=df1[label],
            y=df1[value],
            offsetgroup=0,
            hoverinfo="text",
            text=df1["VALUE_D"],
            hovertext=df1["TEXT"],
            marker=dict(color="#33475b")
        ),
        secondary_y=False,
    )
    
    df2 = df[df[group] == "Amount"].reset_index(drop=True)[:]
    total_value = "{:,.0f}".format(df2[value].sum()).replace(",", " ")
    var_value = df2.loc[df2.index[-1], varv]
    positive = False
    if var_value > 0:
        positive = True
    var_value = "{:,.0f}".format(var_value).replace(",", " ")
    if positive:
        var_value = f"+{var_value}"
    fig.add_trace(
        go.Bar(
            name="Amount",
            x=df2[label],
            y=df2[value],
            text=df2["VALUE_D"] + " K€",
            offsetgroup=1,
            hoverinfo="text",
            hovertext=df2["TEXT"],
            marker=dict(color="#ff7a59")
        ),
        secondary_y=True,
    )

    # Add figure title
    fig.update_layout(
        title=f"<b>Hubspot - New deals created per week</b><br><span style='font-size: 14px;'>Total deals: {total_volume} ({total_value} K€) | This week: {var_volume} ({var_value} K€) vs last week</span>",
        title_font=dict(family="Arial", size=20, color="black"),
        legend=None,
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title="Weeks",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
    )

    # Set y-axes titles
    fig.update_yaxes(
        title_text="No of deals",
        title_font=dict(family="Arial", size=11, color="black"),
        secondary_y=False
    )
    fig.update_yaxes(
        title_text="Amount in K€",
        title_font=dict(family="Arial", size=11, color="black"),
        secondary_y=True
    )
    fig.show()
    return fig

fig = create_barchart(df_trend, "LABEL", "GROUP", "VALUE", "VARV", "VARP")
```

### Export and share graph


```python
# Export in HTML
df_trend.to_csv(csv_output, index=False)
fig.write_image(image_output)
fig.write_html(html_output)

# Shave with naas
csv_url = naas.asset.add(csv_output)
image_url = naas.asset.add(image_output)
html_url = naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below (by removing the hashtag)  to delete your asset
# naas.asset.delete(csv_output)
# naas.asset.delete(image_output)
# naas.asset.delete(html_output)
```

### Create email template


```python
def create_email():
    content = {
        "header": emailbuilder.image(src="https://lib.umso.co/lib_sluGpRGQOLtkyEpz/na1lz0v9ejyurau2.png?w=1200&h=900&fit=max&dpr=2",
                                     link="https://www.hubspot.com/",
                                     align="center",
                                     width="100%"),
        "txt_0": emailbuilder.text("Hi !<br><br>"
                                   f"Here below your weekly update on new deal created as of {datetime.now().strftime('%Y-%m-%d')} :<br>"),
        "image": emailbuilder.image(src=image_url,
                                    link=html_url,
                                    align="center",
                                    width="100%"),
        "button_1": emailbuilder.button(link="https://app.hubspot.com",
                                        text="Go to HubSpot",
                                        color="white",
                                        background_color="#ff7a59"),
        "txt_4": ("Interested to improve this template, please contact the Naas Core Team at hello@naas.ai.<br><br>"),
        "heading_5": emailbuilder.text("Let's close those opportunities 💸!"),
        "footer": emailbuilder.footer_company(naas=True)
    }
    return emailbuilder.generate(display='iframe', **content)

email_content = create_email()
```

## Output

### Send email


```python
naas.notification.send(EMAIL_TO,
                       EMAIL_SUBJECT,
                       email_content)
```
