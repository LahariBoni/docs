<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Get_polls_from_post.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Get+polls+from+post:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #post #polls #naas_drivers #content #analytics #image #html #plotly

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import linkedin
import plotly.express as px
```

### Get your cookies
<a href='https://www.notion.so/LinkedIn-driver-Get-your-cookies-d20a8e7e508e42af8a5b52e33f3dba75'>How to get your cookies ?</a>


```python
LI_AT = 'YOUR_COOKIE_LI_AT'  # EXAMPLE AQFAzQN_PLPR4wAAAXc-FCKmgiMit5FLdY1af3-2
JSESSIONID = 'YOUR_COOKIE_JSESSIONID'  # EXAMPLE ajax:8379907400220387585
```

### Enter post URL


```python
POST_URL = "POST_URL"
```

## Model

### Get poll results from post


```python
df = linkedin.connect(LI_AT, JSESSIONID).post.get_polls(POST_URL)
print("📝 Nb of poll results", len(df))
df.head(5)
```

## Output

### Save your result in csv


```python
df.to_csv("POLL.csv", index=False)
print("💾 Poll results saved in csv")
```

### Create, export and share your graph


```python
def create_polls_graph(df):
    poll_id = df.POLL_ID.unique()[0]
    title = df.POLL_QUESTION.unique()[0]
    
    # Create dataframe
    df = df.groupby(["POLL_RESULT"], as_index=False).agg({"PROFILE_ID": "count"})
    df["VALUE"] = df["PROFILE_ID"] / df["PROFILE_ID"].sum() * 100
    df["VALUE_D"] = df["VALUE"].map('{:.0f}%'.format)
    
    # Count voters
    voters = df.PROFILE_ID.sum()
    
    # Create fig
    fig = px.bar(df,
                 y="POLL_RESULT",
                 x="PROFILE_ID",
                 orientation='h',
                 title=f"{title}<br><span style='font-size: 13px;'>Total amount of votes: {voters}</span>",
                 text="VALUE_D",
                 labels={
                     "POLL_RESULT": "Options",
                     "PROFILE_ID": "Nb of votes",
                     "VALUE_D": "% of votes"
                 },
                 )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        plot_bgcolor="#ffffff",
        width=600,
        height=400,
        font=dict(family="Arial", size=14, color="black"),
        paper_bgcolor="white",
        xaxis_title=None,
        xaxis_showticklabels=False,
        yaxis_title=None,
        margin_pad=10,
    )
    fig.write_html(f"{poll_id}.html")
    fig.show()
    asset = naas.asset.add(f"{poll_id}.html")
    return asset

create_polls_graph(df)
```
