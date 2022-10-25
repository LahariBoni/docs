<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Gmail/Gmail_Send_emails_from_Gsheet_classic.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Gmail+-+Send+emails+from+Gsheet+classic:+Error+short+description">🚨 Bug report</a>

**Tags:** #gmail #productivity #gsheet #naas_drivers #operations #snippet #email

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

Example : to a list of people in Gmail.

## Input

### Import libraries


```python
import naas_drivers
from naas_drivers import gsheet
from naas_drivers import html
```

### Read the gsheet


```python
spreadsheet_id = "1swGTMX6d_N8-AVRueBEd8C0J6OlvO218iDSVMootWZk"
data = naas_drivers.gsheet.connect(spreadsheet_id).get(sheet_name="Sheet1")
```

### Setting your email address


```python
your_email = "jeremy.ravenel@cashstory.com"
firstname_list = data['FIRST NAME']
email_list = data['EMAIL']
```

## Model

### Mail preview


```python
import naas
url_image = naas.assets.add("2020.gif")
email_content = naas_drivers.html.generate( 
        display='iframe',
        title='🎅 Merry Christmas',
        heading= '& Happy new year {first_name} 🍾',
        image = f"{url_image}",
        text_1= "Even if 2020 has been extremely difficult year, let's make 2021 better!",
        text_2= "Keep smiling,",
        text_3= "Keep laughing,",
        text_4= "Spread love ❤️",
        
)
```

## Output

### Sending emails


```python
for i in range(len(data)):
    subject = "Merry Christmas & spread love for 2021 ❤️"
    content = email_content.replace("{first_name}",firstname_list[i])
    naas.notifications.send(email_to=email_list[i], subject=subject, html=content, email_from=your_email)
```
