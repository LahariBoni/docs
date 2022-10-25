<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Emailbuilder_demo.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Emailbuilder+demo:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #emailbuilder #snippet #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
import naas_drivers
import naas
import pandas as pd
```

### Variables


```python
# List to emails address of the receiver(s)
email_to = [""]

# Email sender : Can only take your email account or notification@naas.ai
email_from = ""

# Email subject
subject = "My Object"
```

## Model

### Build the email


```python
table = pd.DataFrame({
    "Table Header 1": ["Left element 1", "Left element 2", "Left element 3"],
    "Table Header 2": ["Right element 1", "Right element 2", "Right element 3"]
})

link = "https://www.naas.ai/"

img = "https://gblobscdn.gitbook.com/spaces%2F-MJ1rzHSMrn3m7xaPUs_%2Favatar-1602072063433.png?alt=media"

list_bullet = ["First element",
               "Second element",
               "Third element",
               naas_drivers.emailbuilder.link(link, "Fourth element"),
              ]

footer_icons = [{
    "img_src": img,
    "href": link
}]

email_content = {
    'element': naas_drivers.emailbuilder.title("This is a title"),
    'heading': naas_drivers.emailbuilder.heading("This is a heading"),
    'subheading': naas_drivers.emailbuilder.subheading("This is a subheading"),
    'text': naas_drivers.emailbuilder.text("This is a text"),
    'link': naas_drivers.emailbuilder.link(link, "This is a link"),
    'button': naas_drivers.emailbuilder.button(link, "This is a button"),
    'list': naas_drivers.emailbuilder.list(list_bullet),
    'table': naas_drivers.emailbuilder.table(table, header=True, border=True),
    'image': naas_drivers.emailbuilder.image(img),
    'footer': naas_drivers.emailbuilder.footer_company(networks=footer_icons, company=["Company informations"], legal=["Legal informations"])
}
```


```python
content = naas_drivers.emailbuilder.generate(display='iframe',
                              **email_content)
```

## Output

### Send the email


```python
naas.notification.send(email_to=email_to,
                       subject=subject,
                       html=content,
                       email_from=email_from)
```
