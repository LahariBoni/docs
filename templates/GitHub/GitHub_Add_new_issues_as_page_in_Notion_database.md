<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/GitHub/GitHub_Add_new_issues_as_page_in_Notion_database.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=GitHub+-+Add+new+issues+as+page+in+Notion+database:+Error+short+description">🚨 Bug report</a>

**Tags:** #github #notion #operations #automation

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)

This notebook is used to create Notion page in a database when a new Github issue is created.
<br/>References :
- Github SDK: [https://github.com/PyGithub/PyGithub](https://github.com/PyGithub/PyGithub)

## Input

### Import libraries


```python
import naas
from naas_drivers import notion, github
```

### Setup GitHub
**How to find your personal access token on GitHub?**

- First we need to create a personal access token to get the details of our organization from here: https://github.com/settings/tokens
- You will be asked to select scopes for the token. Which scopes you choose will determine what information and actions you will be able to perform against the API.
- You should be careful with the ones prefixed with write:, delete: and admin: as these might be quite destructive.
- You can find description of each scope in docs here (https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps).


```python
# GitHub token
GITHUB_TOKEN = "ENTER_YOUR_GITHUB_TOKEN_HERE" # EXAMPLE : "ghp_fUYP0Z5i29AG4ggX8owctGnHU**********" 

# Github repo on which we want to create issues.
REPO_URL = "ENTER_YOUR_REPO_URL_HERE" # EXAMPLE : "https://github.com/jupyter-naas/awesome-notebooks/"
```

### Setup Notion
<a href='https://docs.naas.ai/drivers/notion'>How to get your Notion integration token ?</a>


```python
# Notion token
NOTION_TOKEN = "ENTER_YOUR_NOTION_TOKEN_HERE" # EXAMPLE : "secret_xxskqjlodshfiqs"

# Notion database URL
DATABASE_URL = "https://www.notion.so/naas-official/f42d6592949axxxxxxxxxxxxx" # EXAMPLE : "https://www.notion.so/naas-official/f42d6592949axxxxxxxxxxxxx" 
```

### Setup Naas scheduler


```python
# Schedule your notebook every 15 minutes.
naas.scheduler.add(cron="*/15 * * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()
```

## Model


### Query Notion database


```python
notion_database_id = DATABASE_URL.split("/")[-1].split("?v=")[0]
db_notion = notion.connect(NOTION_TOKEN).database.get(notion_database_id)
df_notion = db_notion.df()

for idx, row in df_notion.iterrows():
    if row.Name=='':
        df_notion.drop(index= idx, inplace=True)
    try:
        df_notion.drop(columns=['Tags'], inplace=True)
    except KeyError:
        pass

df_notion
```

### Get GitHub issues


```python
df_issues = github.connect(GITHUB_TOKEN).repos.get_issues(REPO_URL)
df_issues.head()
```

### Create Notion page from GitHub issue


```python
def notion_page_from_gh_issue(issue, notion_database_id):
    if len(df_notion) == 0:
        pass
    elif len(df_notion) != 0 and str(issue.issue_id) in df_notion.Issue_id.to_list():
        return "issue already exists in database"
    
    page = notion.connect(NOTION_TOKEN).page.create(database_id=notion_database_id,
                                                    title=issue.issue_title)
    page.rich_text("URL",issue.link_to_the_issue)
    page.rich_text("Assignees",issue.issue_assignees)
    page.number('Issue_id',issue.issue_id)
    page.date('Last_created',str(issue.last_created_date)+'T'+str(issue.last_created_time))
    page.update()
    return "Done"
```

## Output


### Iterate over each issue.



```python
for idx, issue in df_issues.iterrows():
    val = notion_page_from_gh_issue(issue, notion_database_id)
    if val == "issue already exists in database":
        print("Database up to date!")
        break
    print(f'✅ Notion page created for issue {issue.link_to_the_issue}')
```
