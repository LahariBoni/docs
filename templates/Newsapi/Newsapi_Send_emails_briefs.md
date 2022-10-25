<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Newsapi/Newsapi_Send_emails_briefs.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Newsapi+-+Send+emails+briefs:+Error+short+description">🚨 Bug report</a>

**Tags:** #newsapi #news #emailbrief #automation #notification #opendata #email #image #html #text

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

Source: https://newsapi.org/

## Input

### Import libraries


```python
from naas_drivers import newsapi, emailbuilder
import naas
```

### Setup your variables


```python
# Input
query = "data, automation, AI" #newsapi query

# Outputs
your_email = "*********"
email_subject = "News scheduled from Naas dev"
email_from = 'notifications@naas.ai'
```

### Schedule everyday at 8am CET


```python
naas.scheduler.add(recurrence="0 8 * * *")

# Uncomment the line below and run the cell to delete your scheduler
# naas.scheduler.delete()
```

## Model

### Use NewsAPI drivers to get data


```python
table_news = newsapi.connect().get(q=query)

# rename columns match the field required by Naas emailbuilder drivers
table_news.rename(columns={'title':'text'}, inplace=True)
table_news.rename(columns={'link':'row_link'}, inplace=True)
```

### Filter results 


```python
table_news_email = table_news[:10]
table_news_email = table_news_email[['text','row_link']]
table_news_email
```

### Format HTML content


```python
links = []
ht_str = "<ul>"
for i in range(len(table_news_email)):
    val = "<li>"+"<a href="+'"'+table_news_email['row_link'][i]+'"'+">"+table_news_email['text'][i]+"</a>"+"</li>"
    ht_str = ht_str+'\n'+val
ht_str = ht_str+"\n"+"</ul>" 
```

### Fill the content of the mail


```python
email_content = emailbuilder.generate( 
        display='iframe',
        title=f'🌏 NewsAPI brief', 
        subtitle=f'<b>Topics</b>: {query}',         
        table_1= ht_str,
        text="Source: <a>https://newsapi.org/</a>"
        )
```

## Output

### Send email


```python
naas.notification.send(email_to=your_email,
                       subject=email_subject,
                       html=email_content,
                       email_from=email_from)
```
