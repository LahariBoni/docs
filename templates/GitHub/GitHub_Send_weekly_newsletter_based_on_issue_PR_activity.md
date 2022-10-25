<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Send_weekly_newsletter_based_on_issue_PR_activity.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Send+weekly+newsletter+based+on+issue+PR+activity:+Error+short+description">🚨 Bug report</a>

**Tags:** #tool #naas_drivers #naas #scheduler #asset #snippet #automation #ai #newsletter

**Author:** [Suhas B](https://www.linkedin.com/in/suhasbrao/)

The notebook allows you to create a weekly newsletter based on issue/PR activity for a GitHub repository.

## Input

### Import libraries


```python
from naas_drivers import github
import naas
import markdown2
from IPython.core.display import display, HTML
import datetime
import pandas as pd
import requests
from urllib.parse import urlencode
```

### Setup GitHub

Before running the below cell ensure you have added a GitHub access token for repository using naas 
by running **naas.secret.add(name="API_NAME", secret="API_KEY")** and delete that cell.

You can find more info [here](https://docs.naas.ai/features/secret#add-or-edit-secret)


```python
# GitHub credentials
GITHUB_TOKEN = naas.secret.get(name="GITHUB_TOKEN")

# Repository
repo_url = "https://github.com/jupyter-naas/awesome-notebooks"

# Number of past days to follow activity
no_past_days = 7
```

### Setup Naas notifications


```python
# List of email_receivers
email_receivers = ["hello@naas.ai"] 

# EMAIL_FROM = None
EMAIL_SUBJECT = "✉️ Weekly Repository activity"

# Markdown template created on your local env
EMAIL_CONTENT_MD = "email_content.md"
```

### Schedule your notebook

Scheduling notebook to run At 12:00 on Sunday. You can modify it if you want 


```python
# Schedule your notebook 
naas.scheduler.add(cron="0 12 * * 0")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()
```

## Model

### Get Open issues from GitHub repository


```python
# Get the open issues from github naas_drivers
df_issues = github.connect(GITHUB_TOKEN).repos.get_issues(repo_url)
print("Total Issues fetched:", len(df_issues))
```

### Get closed issues


```python
def get_closed_issues(repo_url):
    '''
    This function retrives the closed issues of a repository and returns a 
    dataframe with the following columns:
    This function retrives the closed PRs of a repository and returns a 
    dataframe with the following columns:
    link_to_the_issue	
    issue_number	
    issue_title	
    issue_state	
    issue_assignees	
    last_created_date	
    last_created_time	
    last_updated_date	
    last_updated_time	
    link_to_the_pr	
    linked_pr_state	
    PR_activity

    Parameters
    ----------
    repository: str:
        Repository url from Github.
        Example : "https://github.com/jupyter-naas/awesome-notebooks"
    '''
    
    git_obj = github.connect(GITHUB_TOKEN)
    repository = git_obj.get_repository_url(repo_url)
    df = pd.DataFrame()
    page, idx = 1, 0
    while True:
        params = {
                "per_page": "100",
                "page": page,
            }
        
        url = f"https://api.github.com/repos/{repository}/issues?state=closed&{urlencode(params, safe='(),')}"
        res = requests.get(url, headers=git_obj.headers)
        try:
            res.raise_for_status()
        except requests.HTTPError as e:
            raise(e)
        
        res_json = res.json()
        if len(res_json) == 0:
            break
        for issue in res_json:
            df.loc[idx,'link_to_the_issue'], df.loc[idx, 'issue_number'] = issue['html_url'], issue['number']
            df.loc[idx, 'issue_title'], df.loc[idx, 'issue_state'] = issue['title'], issue['state']
#             df.loc[idx, 'issue_id'] = issue['id']
            assigned=[]
            for assignee in issue['assignees']:
                assigned.append(assignee.get('login'))
            if assigned==[]:
                df.loc[idx, 'issue_assignees'] = 'None'
            else:
                df.loc[idx, 'issue_assignees'] = ", ".join(assigned)
            
            df.loc[idx, 'last_created_date'] = issue.get('created_at').strip('Z').split('T')[0]
            df.loc[idx, 'last_created_time'] = issue.get('created_at').strip('Z').split('T')[-1]
            df.loc[idx, 'last_updated_date'] = issue.get('updated_at').strip('Z').split('T')[0]
            df.loc[idx, 'last_updated_time'] = issue.get('updated_at').strip('Z').split('T')[-1]
            try:
#                 print(issue.get('pull_request')['url'])
                df.loc[idx, "link_to_the_pr"] = issue.get('pull_request')['html_url']
                pr = requests.get(issue.get('pull_request')['url'], headers= git_obj.headers).json()
                
                df.loc[idx, 'linked_pr_state'] = pr.get('state')

                date_format = "%Y-%m-%d"
                delta = datetime.datetime.now() - datetime.datetime.strptime(df.loc[idx, 'last_updated_date'], date_format)
                df.loc[idx, 'PR_activity'] = f'No activity since {delta.days} days'

            except Exception as e:
                #print(e)
                df.loc[idx, 'linked_pr_state'] = 'None'
                df.loc[idx, 'PR_activity'] = 'None'
            idx +=1
        page+=1
    return df

df_closed_issues = get_closed_issues(repo_url)
print("Total closed Issues fetched:", len(df_closed_issues))
```

### Get open PRs


```python
def get_open_PRs(repo_url):
        """
        Return an dataframe object with 15 columns:
        - ID                      int64
        - ISSUE_URL               object
        - PR_NUMBER               int64
        - PR_STATE                object
        - TITLE                   object
        - ISSUE_REF_TITILE        object
        - ASSIGNEES_PROFILE       object
        - FIRST_CREATED_DATE      object
        - FIRST_CREATED_TIME      object
        - LAST_UPDATED_DATE       object
        - LAST_UPDATED_TIME       object
        - COMMITS_URL             object
        - REVIEW_COMMENTS_URL     object
        - ISSUE_COMMENTS_URL      object
        - ASSIGNEES               object
        - REQUESTED_REVIEWERS     object
        - PR_ACTIVITY             object
        Parameters
        ----------
        repository: str:
            Repository url from Github.
            Example : "https://github.com/jupyter-naas/awesome-notebooks"
        """
        # Get organisation and repository from url
        git_obj = github.connect(GITHUB_TOKEN)
        repository = git_obj.get_repository_url(repo_url)
        
        df = pd.DataFrame()
        page = 1
        while True:
            params = {
                "per_page": "100",
                "page": page,
            }
            url = f"https://api.github.com/repos/{repository}/pulls?&{urlencode(params, safe='(),')}"
            res = requests.get(url, headers=git_obj.headers)
            try:
                res.raise_for_status()
            except requests.HTTPError as e:
                raise(e)
            res_json = res.json()
            if len(res_json) == 0:
                break

            for idx, r in enumerate(res_json):
                if r.get('state') == 'open':
                    df.loc[idx, 'id'] = r.get('id')
                    df.loc[idx, 'issue_url'] = r.get('html_url')
                    df.loc[idx, 'PR_number'] = r.get('number')
                    df.loc[idx, 'PR_state'] = 'open'
                    df.loc[idx, 'Title'] = r.get('title')
                    
                    df.loc[idx, 'issue_ref_title'] = r.get('head').get('ref')
                    
                    df.loc[idx, 'first_created_date'] = r.get('created_at').strip('Z').split('T')[0]
                    df.loc[idx, 'first_created_time'] = r.get('created_at').strip('Z').split('T')[-1]
                    df.loc[idx, 'last_updated_date'] = r.get('updated_at').strip('Z').split('T')[0]
                    df.loc[idx, 'last_updated_time'] = r.get('updated_at').strip('Z').split('T')[-1]

                    df.loc[idx, 'commits_url'] = r.get('commits_url')
                    df.loc[idx, 'review_comments_url'] = r.get('review_comments_url')
                    df.loc[idx, 'issue_comments_url'] = r.get('comments_url')

                    assignees_lst, reviewers_lst=[],[]
                    assignees_profile = []
                    for assignee in r.get('assignees'):
                        assignees_lst.append(assignee.get('login'))
                        assignees_profile.append(assignee.get('html_url'))
                    for reviewer in r.get('requested_reviewers'):
                        reviewers_lst.append(reviewer.get('login'))

                    if assignees_lst==[]:
                        df.loc[idx, 'assignees'] = 'None'
                        df.loc[idx, 'assignees_profile'] = 'None'
                    elif assignees_lst:
                        df.loc[idx, 'assignees'] = ", ".join(assignees_lst)
                        df.loc[idx, 'assignees_profile'] = ", ".join(assignees_profile)

                    if reviewers_lst==[]:
                        df.loc[idx, 'requested_reviewers'] = 'None'
                    elif reviewers_lst:
                        df.loc[idx, 'requested_reviewers'] = ", ".join(reviewers_lst)

                    date_format = "%Y-%m-%d"
                    delta = datetime.datetime.now() - datetime.datetime.strptime(df.loc[idx, 'last_updated_date'], date_format)
                    df.loc[idx, 'PR_activity'] = f'No activity since {delta.days} days'

                df['PR_number'] = df.PR_number.astype('int')
                df.id = df.id.astype('int')
            page+=1
        return df
    
df_open_PRS = get_open_PRs(repo_url)
print("Total open PRs:", len(df_open_PRS))
```

### Get closed PRs


```python
def get_closed_PRs(repo_url):
    '''
    This function retrives the closed PRs of a repository and returns a 
    dataframe with the following columns:
        - ID                      int64
        - ISSUE_URL               object
        - PR_NUMBER               int64
        - PR_STATE                object
        - TITLE                   object
        - FIRST_CREATED_DATE      object
        - FIRST_CREATED_TIME      object
        - LAST_UPDATED_DATE       object
        - LAST_UPDATED_TIME       object
        - COMMITS_URL             object
        - REVIEW_COMMENTS_URL     object
        - ISSUE_COMMENTS_URL      object
        - ASSIGNEES               object
        - REQUESTED_REVIEWERS     object
        - REVIEWERS_PROFILE       object
        - PR_ACTIVITY             object
        Parameters
        ----------
        repository: str:
            Repository url from Github.
            Example : "https://github.com/jupyter-naas/awesome-notebooks"
    '''
    
    git_obj = github.connect(GITHUB_TOKEN)
    repository = git_obj.get_repository_url(repo_url)    
    df = pd.DataFrame()
    page, idx = 1, 0
#     page = 1
    while True:
            params = {
                "per_page": "200",
                "page": page,
            }
            url = f"https://api.github.com/repos/{repository}/pulls?state=closed&{urlencode(params, safe='(),')}"
            res = requests.get(url, headers=git_obj.headers)
            try:
                res.raise_for_status()
            except requests.HTTPError as e:
                raise(e)
            res_json = res.json()
            if len(res_json) == 0:
                break

            for _,r in enumerate(res_json):
#                 if r.get('state') == 'closed':
                    df.loc[idx, 'id'] = r.get('id')
                    df.loc[idx, 'issue_url'] = r.get('html_url')
                    df.loc[idx, 'PR_number'] = r.get('number')
                    df.loc[idx, 'PR_state'] = 'open'
                    df.loc[idx, 'Title'] = r.get('title')

                    df.loc[idx, 'first_created_date'] = r.get('created_at').strip('Z').split('T')[0]
                    df.loc[idx, 'first_created_time'] = r.get('created_at').strip('Z').split('T')[-1]
                    df.loc[idx, 'last_updated_date'] = r.get('updated_at').strip('Z').split('T')[0]
                    df.loc[idx, 'last_updated_time'] = r.get('updated_at').strip('Z').split('T')[-1]

                    df.loc[idx, 'commits_url'] = r.get('commits_url')
                    df.loc[idx, 'review_comments_url'] = r.get('review_comments_url')
                    df.loc[idx, 'issue_comments_url'] = r.get('comments_url')

                    assignees_lst, reviewers_lst=[],[]
                    reviewers_profile = []
                    for assignee in r.get('assignees'):
                        assignees_lst.append(assignee.get('login'))
                    for reviewer in r.get('requested_reviewers'):
                        reviewers_lst.append(reviewer.get('login'))
                        reviewers_profile.append(reviewer.get('html_url'))

                    if assignees_lst==[]:
                        df.loc[idx, 'assignees'] = 'None'
                    elif assignees_lst:
                        df.loc[idx, 'assignees'] = ", ".join(assignees_lst)

                    if reviewers_lst==[]:
                        df.loc[idx, 'requested_reviewers'] = 'None'
                        df.loc[idx, 'reviewers_profile'] = 'None'
                    elif reviewers_lst:
                        df.loc[idx, 'requested_reviewers'] = ", ".join(reviewers_lst)
                        df.loc[idx, 'reviewers_profile'] = ", ".join(reviewers_profile)

                    date_format = "%Y-%m-%d"
                    delta = datetime.datetime.now() - datetime.datetime.strptime(df.loc[idx, 'last_updated_date'], date_format)
                    df.loc[idx, 'PR_activity'] = f'No activity since {delta.days} days'

                    df['PR_number'] = df.PR_number.astype('int')
                    df.id = df.id.astype('int')
                    idx +=1
            page+=1
    return df
df_closed_prs = get_closed_PRs(repo_url)
print("Total closed PRs:", len(df_closed_prs))
```

### Get KPIs


```python
# Get the latest issues between previous_date and current_date
# This is acheived by using a mask as shown below
current_date = datetime.date.today()
previous_date = datetime.datetime.today() - datetime.timedelta(days= no_past_days)
previous_date = previous_date.date()
no_issues_created = 0
no_issues_closed = 0

mask_for_open_issues = (df_issues["last_created_date"] >= str(previous_date)) & \
                        (df_issues["last_created_date"] <= str(current_date))
# mask.head(5)

mask_for_closed_issues = (df_closed_issues["last_created_date"] >= str(previous_date)) & \
                        (df_closed_issues["last_created_date"] <= str(current_date))


mask_for_closed_PRs = (df_closed_prs["last_updated_date"] >= str(previous_date)) & \
                      (df_closed_prs["last_updated_date"] <= str(current_date))

mask_for_open_PRs = (df_open_PRS["last_updated_date"] >= str(previous_date)) & \
                      (df_open_PRS["last_updated_date"] <= str(current_date))


df_new_issues = df_issues.loc[mask_for_open_issues]
df_closed_issues_needed = df_closed_issues.loc[mask_for_closed_issues]
df_closed_PRs_needed = df_closed_prs.loc[mask_for_closed_PRs]
df_open_PRs_needed = df_open_PRS.loc[mask_for_open_PRs]

no_issues_created = len(df_new_issues)
no_issues_closed = len(df_closed_issues_needed)
no_PRs_closed = len(df_closed_PRs_needed)
no_PRs_open = len(df_open_PRs_needed)
```

### Get issues for open PRs


```python
def get_issue_objs_from_prs(df_opens, df_open_PRs_needed):
    df = pd.DataFrame()
    for values in df_open_PRs_needed["issue_url"]:

        df_new = df_opens[df_opens['link_to_the_issue'] == values]
        df = df.append(df_new, ignore_index = True)
    return df

df_issues_for_opens_prs = get_issue_objs_from_prs(df_issues, df_open_PRs_needed)
df_issues_for_opens_prs
```

### Create a content for mail
This is later to converted to a HTML content for the mail. Modify these contents based on your needs.


```python
def show_open_issues(open_issues_required) -> str:
#     BigFunctions - Get sentiment score #1240
    open_issues_required = open_issues_required.reset_index()
    
    temp_str = ""
    for row in range(len(open_issues_required)):
    
        line = f'''- [{df_new_issues["issue_title"][row]} #{open_issues_required["issue_number"][row]}]({df_new_issues["link_to_the_issue"][row]})
        '''
        temp_str += '\n' + line

    return temp_str

def show_closed_issues(closed_issues_required, closed_prs_needed) -> str:
    closed_issues_required = closed_issues_required.reset_index()
    closed_prs_needed = closed_prs_needed.reset_index()
    temp_str = ""
    for row in range(len(closed_issues_required)):
    
        line = f'''- [ {closed_issues_required["issue_title"][row]} \
        {int(closed_issues_required["issue_number"][row])}]({closed_issues_required["link_to_the_issue"][row]}),
    closed by #[{closed_prs_needed["PR_number"][row]}]({closed_prs_needed["issue_url"][row]}) \
    by [@{closed_prs_needed["requested_reviewers"][row]}]({closed_prs_needed["reviewers_profile"][row]})
    '''
        
        temp_str += '\n' + line

    return temp_str
    

def show_open_PRs(open_prs_req, issues_for_open_prs) -> str:
    open_prs_req = open_prs_req.reset_index()
    temp_str = ""
    df_1 = open_prs_req
    df_2 = issues_for_open_prs
    for row in range(len(open_prs_req)):

        date_str = df_1["first_created_date"][row]+' '+df_1["first_created_time"][row]
        date_created = datetime.datetime.strptime(date_str, "%Y-%m-%d %H:%M:%S")
        no_days_ago = (datetime.date.today() - date_created.date()).days 
        
        line = f'''- [#{df_1["PR_number"][row]}]({df_1["issue_url"][row]}), solving Issue \
        [**{df_2["issue_title"][row].capitalize()} #{df_2["issue_number"][row]}**]({df_2["link_to_the_issue"][row]}) from \
        @[{df_1["assignees"][row]}]({df_1["assignees_profile"][row]}) opened \
        {no_days_ago} days ago ({df_1["first_created_date"][row]}) 
        
        '''
        temp_str += '\n' + line

    return temp_str
```


```python
# content = open(EMAIL_CONTENT_MD, "r").read()

content = f'''
Hello there 👋,

Here is what happened over the past {no_past_days} days on **{repo_url.split("/")[-1]}**.


### Issues created = {no_issues_created}

{show_open_issues(df_new_issues)}

### Issues closed = {no_issues_closed}

{show_closed_issues(df_closed_issues_needed, df_closed_PRs_needed)}

### PRs Open = {no_PRs_open}

{show_open_PRs(df_open_PRs_needed, df_issues_for_opens_prs)}


Check out the latest issues activity on [GitHub]({repo_url})⭐

Cheers 😊
'''

md = markdown2.markdown(content)
# email_content = replace_values(md)
display(HTML(md))
```

## Output

### Send mail to receivers


```python
for receiver in email_receivers:
    naas.notification.send(email_to=receiver,
                           subject=EMAIL_SUBJECT,
                           html=md)
```


```python

```
