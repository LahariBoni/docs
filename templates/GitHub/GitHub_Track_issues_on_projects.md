<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Track_issues_on_projects.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Track+issues+on+projects:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #repos #issues #operations #analytics #csv #plotly

**Author:** [Sanjeet Attili](https://www.linkedin.com/in/sanjeet-attili-760bab190/)

The objective of the notebook is to maintain a track of issues that are open in the community roadmap

## Input

### Imports libraries


```python
import plotly.express as px
from naas_drivers import github
```

### Setup GitHub
**How to find your personal access token on Github?** 
- First we need to create a personal access token to get the details of our organization from here: https://github.com/settings/tokens
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API. 
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive. 
- You can find description of each scope in docs here (https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


```python
# Project URL from web
PROJECT_URL = "https://github.com/orgs/jupyter-naas/projects"

# GitHub Token
GITHUB_TOKEN = "ghp_COJiJEU4cQR4rjsxxxxxx"
```

### Setup variables


```python
# Graph : bar order
STATUS_ORDER = ["Backlog", "To Do", "In Progress", "Review", "Done"]

# Outputs
csv_output = "GitHub_Issues.csv"
```

## Model

### Get issues from project on Github


```python
df_issues = github.connect(GITHUB_TOKEN).projects.get_issues(PROJECT_URL)
df_issues.tail(15)
```

### Create barchart


```python
for index, s in enumerate(STATUS_ORDER):
    print(index, s)
```


```python
def create_barchart(df, title, labels):
    # Get status
    status = df.issue_status.unique().tolist()
    status_order = []
    for index, s in enumerate(STATUS_ORDER):
        if s in status:
            status_order += [s]
    
    # Create fig
    fig = px.bar(df,
                 title=title,
                 x=status_order,
                 y="count",
                 text="count",
                 labels=labels)
    fig.update_traces(marker_color='black')
    fig.update_layout(
        plot_bgcolor="#ffffff",
        width=1000,
        height=800,
        font=dict(family="Arial", size=14, color="black"),
        paper_bgcolor="white",
        yaxis_title="No of issues",
        yaxis_title_font=dict(family="Arial", size=11, color="black"),
        xaxis_title="Status",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig
```

## Output

### Save data in csv


```python
df_issues.to_csv(csv_output, index=False)
```

### Plotting a bar graph for total number of issues


```python
# Dataframe
issues = df_issues.groupby('issue_status').agg({'issue_number':'count'}).reset_index().rename(columns={"issue_number":"count"})

# Chart title
title =  f"Github Project - {PROJECT_URL.split('/')[-2]} : {df_issues['project_name'].unique()[0]} <br><span style='font-size: 13px;'>Total issues: {issues['count'].sum()}</span>"

# Hover labels
labels = {
    'issue_status':'Issue status',
    'count':"Number of Issues"
}
fig = create_barchart(issues, title, labels)
```

### Plotting a bar graph for open issues


```python
# Dataframe
open_issues = df_issues[df_issues['issue_state']=='open'].groupby('issue_status').agg({"issue_number":'count'}).reset_index().rename(columns={'issue_number':'count'})

# Chart title
title =  f"Github Project - {PROJECT_URL.split('/')[-2]} : {df_issues['project_name'].unique()[0]} <br><span style='font-size: 13px;'>Total open issues: {open_issues['count'].sum()}</span>"

# Hover labels
labels = {
               'issue_status':'Issue status',
               'count':"Number of Open issues"
          }
fig = create_barchart(open_issues, title, labels)
```

### Plotting a bar graph for closed issues


```python
# Dataframe
closed_issues = df_issues[df_issues['issue_state']=='closed'].groupby('issue_status').agg({"issue_number":'count'}).reset_index().rename(columns={'issue_number':'count'})

# Chart title
title =  f"Github Project - {PROJECT_URL.split('/')[-2]} : {df_issues['project_name'].unique()[0]} <br><span style='font-size: 13px;'>Total closed issues: {closed_issues['count'].sum()}</span>"

# Hover labels
labels = {
               'issue_status':'Issue status',
               'count':"Number of Closed issues"
          }

fig = create_barchart(closed_issues, title, labels)
```

### Plotting a bar graph for stale issues


```python
stale_issues = []
for item in df_issues.stale_issue:
    if item!='None':
        stale_issues.append(int(item.split()[-2])>=7)
    else:
         stale_issues.append(False)
            
df_issues['stale_bool'] = stale_issues
temp = df_issues[df_issues['stale_bool']==True]
temp[temp['issue_state']=='open']

# Dataframe
open_stale_issues = temp[temp['issue_state']=='open'].groupby('issue_status').agg({"stale_bool":'count'}).reset_index().rename(columns={'stale_bool':'count'})

# Chart title
title =  f"Github Project - {PROJECT_URL.split('/')[-2]} : {df_issues['project_name'].unique()[0]} <br><span style='font-size: 13px;'>Total open stale issues: {open_stale_issues['count'].sum()}</span>"

# Hover labels
labels = {
               'issue_status':'Issue status',
               'count':"Number of Open issues with no activity since more than 7 days"
          }
fig = create_barchart(open_stale_issues, title, labels)
```
