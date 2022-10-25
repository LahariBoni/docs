<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/HubSpot/HubSpot_Send_sales_brief.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=HubSpot+-+Send+sales+brief:+Error+short+description">🚨 Bug report</a>

**Tags:** #hubspot #crm #sales #deal #naas_drivers #notification #asset #emailbuilder #scheduler #naas #analytics #automation #email #text #plotly #html #image

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import emailbuilder, hubspot
import naas
import pandas as pd
from datetime import datetime
```

### Setup your HubSpot
👉 Access your [HubSpot API key](https://knowledge.hubspot.com/integrations/how-do-i-get-my-hubspot-api-key)


```python
HS_API_KEY = 'YOUR_HUBSPOT_API_KEY'
```

### Email parameters


```python
# Receivers
email_to = ["your_email_adresse"]

# Email subject
email_subject = f"🚀 Hubspot - Sales Brief as of {datetime.now().strftime('%d/%m/%Y')} (Draft)"
```

### Sales target


```python
objective = 300000
```

### Pick your pipeline

#### Get all pipelines


```python
df_pipelines = hubspot.connect(HS_API_KEY).pipelines.get_all()
df_pipelines
```

#### Enter your pipeline id


```python
pipeline_id = "8432671"
```

### Constants


```python
HUBSPOT_CARD = "https://lib.umso.co/lib_sluGpRGQOLtkyEpz/na1lz0v9ejyurau2.png?w=1200&h=900&fit=max&dpr=2"
NAAS_WEBSITE = "https://www.naas.ai"
EMAIL_DESCRIPTION = "Your sales brief"
DATE_FORMAT = "%Y-%m-%d"
```

### Schedule automation


```python
#-> Uncomment the 2 lines below (by removing the hashtag) to schedule your job every monday at 8:00 AM (NB: you can choose the time of your scheduling bot)
# import naas
# naas.scheduler.add(cron="0 8 * * 1")

#-> Uncomment the line below (by removing the hashtag) to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get dealstages from pipeline


```python
df_dealstages = df_pipelines.copy()

# Filter on pipeline
df_dealstages = df_dealstages[df_dealstages.pipeline_id == pipeline_id]

df_dealstages
```

### Get deals from pipeline


```python
properties = [
    "hs_object_id",
    "dealname",
    "dealstage",
    "pipeline",
    "createdate",
    "hs_lastmodifieddate",
    "closedate",
    "amount"
]
df_deals = hubspot.connect(HS_API_KEY).deals.get_all(properties)

# Filter on pipeline
df_deals = df_deals[df_deals.pipeline == pipeline_id].reset_index(drop=True)

df_deals
```

### Formatting functions


```python
def format_number(num):
    NUMBER_FORMAT = "{:,.0f} €"
    num = str(NUMBER_FORMAT.format(num)).replace(",", " ")
    return num
```


```python
def format_pourcentage(num):
    NUMBER_FORMAT = "{:,.0%}"
    num = str(NUMBER_FORMAT.format(num))
    return num
```


```python
def format_varv(num):
    NUMBER_FORMAT = "+{:,.0f} €"
    num = str(NUMBER_FORMAT.format(num)).replace(",", " ")
    return num
```

### Create sales pipeline database


```python
df_sales = pd.merge(df_deals.drop("pipeline", axis=1),
                    df_dealstages.drop(["pipeline", "pipeline_id", "createdAt", "updatedAt", "archived"], axis=1),
                    left_on="dealstage",
                    right_on="dealstage_id",
                    how="left")
df_sales
```


```python
df_sales_c = df_sales.copy()

# Cleaning
df_sales_c["amount"] = df_sales_c["amount"].fillna("0")
df_sales_c.loc[df_sales_c["amount"] == "", "amount"] = "0"

# Formatting
df_sales_c["amount"] = df_sales_c["amount"].astype(float)
df_sales_c["probability"] =  df_sales_c["probability"].astype(float)
df_sales_c.createdate = pd.to_datetime(df_sales_c.createdate)
df_sales_c.hs_lastmodifieddate = pd.to_datetime(df_sales_c.hs_lastmodifieddate)
df_sales_c.closedate = pd.to_datetime(df_sales_c.closedate)

# Calc
df_sales_c["forecasted"] = df_sales_c["amount"] * df_sales_c["probability"]

df_sales_c
```

### Create sales pipeline agregated by dealstages


```python
df_details = df_sales_c.copy()

# Groupby
to_group = [
    "dealstage_label",
    "probability",
    "displayOrder"
]
to_agg = {
    "amount": "sum",
    "dealname": "count",
    "forecasted": "sum"
}
df_details = df_details.groupby(to_group, as_index=False).agg(to_agg)

# Sort
df_details = df_details.sort_values("displayOrder")

df_details
```

### Calculate email parameters


```python
forecasted = df_details.forecasted.sum()
forecasted
```


```python
won = df_details[df_details["probability"] == 1].forecasted.sum()
won
```


```python
weighted = df_details[df_details["probability"] < 1].forecasted.sum()
weighted
```


```python
completion_p = forecasted / objective
completion_p
```


```python
completion_v = objective - forecasted
completion_v
```


```python
today = datetime.now().strftime(DATE_FORMAT)
today
```

### Get pipeline details


```python
df = df_details.copy()

details = []

for _, row in df.iterrows():
    # status part
    dealstage = row.dealstage_label
    probability = row.probability
    detail = f"{dealstage} ({format_pourcentage(probability)})"
    
    # amount part
    amount = row.amount
    number = row.dealname
    forecasted_ = row.forecasted
    if (probability < 1 and probability > 0):
        detail = f"{detail}: <ul><li>Amount : {format_number(amount)}</li><li>Number : {number}</li><li>Weighted amount : <b>{format_number(forecasted_)}</b></li></ul>"
    else:
        detail = f"{detail}: {format_number(amount)}"
        
    details += [detail]

details
```

### Get inactives deals


```python
df_inactive = df_sales_c.copy()

df_inactive.hs_lastmodifieddate = pd.to_datetime(df_inactive.hs_lastmodifieddate).dt.strftime(DATE_FORMAT)

df_inactive["inactive_time"] = (datetime.now() - pd.to_datetime(df_inactive.hs_lastmodifieddate, format=DATE_FORMAT)).dt.days
df_inactive.loc[(df_inactive["inactive_time"] > 30, "inactive")] = "inactive"
df_inactive = df_inactive[(df_inactive.inactive == 'inactive') &
                          (df_inactive.amount != 0) & 
                          (df_inactive.probability > 0.) & 
                          (df_inactive.probability < 1)].sort_values("amount", ascending=False).reset_index(drop=True)

df_inactive
```


```python
inactives = []

for _, row in df_inactive[:10].iterrows():
    # status part
    dealname = row.dealname
    dealstage_label = row.dealstage_label
    amount = row.amount
    probability = row.probability
    inactive = f"{dealname} ({dealstage_label}): <b>{format_number(amount)}</b>"
    inactives += [inactive]

inactives
```

### Create pipeline waterfall


```python
import plotly.graph_objects as go

fig = go.Figure(go.Waterfall(name="20",
                             orientation = "v",
                             measure = ["relative", "relative", "total", "relative", "total"],
                             x = ["Won", "Pipeline", "Forecast", "Missing", "Objective"],
                             textposition = "outside",
                             text = [format_number(won), format_varv(weighted), format_number(forecasted), format_varv(completion_v), format_number(objective)],
                             y = [won, weighted, forecasted, completion_v, objective],
                            decreasing = {"marker":{"color":"#33475b"}},
                            increasing = {"marker":{"color":"#33475b"}},
                            totals = {"marker":{"color":"#ff7a59"}}
))


fig.update_layout(title = "Sales Metrics", plot_bgcolor="#ffffff", hovermode='x')
fig.update_yaxes(tickprefix="€", gridcolor='#eaeaea')
fig.show()
```


```python
fig.write_html("GRAPH_FILE.html")
fig.write_image("GRAPH_IMG.png")

params = {"inline": True}

graph_url = naas.asset.add("GRAPH_FILE.html", params=params)
graph_image = naas.asset.add("GRAPH_IMG.png")
```

### Create email


```python
def email_brief(today,
                forecasted,
                won,
                weighted,
                objective,
                completion_p,
                completion_v,
                details,
                inactives
                ):
    content = {
        'title': (f"<a href='{NAAS_WEBSITE}'>"
                   f"<img align='center' width='100%' target='_blank' style='border-radius:5px;'"
                   f"src='{HUBSPOT_CARD}' alt={EMAIL_DESCRIPTION}/>"
                   "</a>"),
        
        'txt_intro': (f"Hi there,<br><br>"
                      f"Here is your weekly sales email as of {today}."),

        'title_1': emailbuilder.text("Overview", font_size="27px", text_align="center", bold=True),
        "text_1": emailbuilder.text(f"As of today, your yearly forecasted revenue is {format_number(forecasted)}."),
        "list_1": emailbuilder.list([f"Won : {format_number(won)}",
                                     f"Weighted pipeline : <b>{format_number(weighted)}</b>"]),
        "text_1_2": emailbuilder.text(f"You need to find 👉 <u>{format_number(completion_v)}</u> to reach your goal !"),
        "text_1_1": emailbuilder.text(f"Your yearly objective is {format_number(objective)} ({format_pourcentage(completion_p)} completion)."),
        'image_1': emailbuilder.image(graph_image, link=graph_url),
        
        'title_2': emailbuilder.text("🚀 Pipeline", font_size="27px", text_align="center", bold=True),
        "list_2": emailbuilder.list(details),

        'title_3': emailbuilder.text("🧐 Actions needed", font_size="27px", text_align="center", bold=True),
        'text_3': emailbuilder.text("Here are deals where you need to take actions :"),
        'list_3': emailbuilder.list(inactives),
        'text_3_1': emailbuilder.text("If you need more details, connect to Hubspot with the link below."),
        'button_1': emailbuilder.button(link="https://app.hubspot.com/",
                                        text="Go to Hubspot",
                                        background_color="#ff7a59"),
        
        'title_4': emailbuilder.text("Glossary", text_align="center", bold=True, underline=True),
        'list_4': emailbuilder.list(["Yearly forecasted revenue :  Weighted amount + WON exclude LOST",
                                     "Yearly objective : Input in script",
                                     "Inactive deal : No activity for more than 30 days"]),
        
        'footer_cs': emailbuilder.footer_company(naas=True),
    }
    
    email_content = emailbuilder.generate(display='iframe', **content)
    return email_content

email_content = email_brief(today,
                            forecasted,
                            won,
                            weighted,
                            objective,
                            completion_p,
                            completion_v,
                            details,
                            inactives)
```

## Output

### Send email


```python
naas.notification.send(email_to,
                       email_subject,
                       email_content)
```
