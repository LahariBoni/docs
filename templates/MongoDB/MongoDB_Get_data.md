<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/MongoDB/MongoDB_Get_data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=MongoDB+-+Get+data:+Error+short+description">🚨 Bug report</a>

**Tags:** #mongodb #database #naas_drivers #snippet #operations #naas

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import library


```python
from naas_drivers import mongo
```

### Setup your MongoDB


```python
# Credentials
user = "my user"
passwd = "my passwd"
host = "url"
port = 9090

# DB parameters
collection_name = "col"
db_name = "db_name"
```

## Model

### Get data


```python
df = naas_drivers.mongo.connect(host, port, user, passwd).get(collection_name, db_name)
```

## Output

### Display result


```python
df
```
