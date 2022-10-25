<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/BigQuery/BigQuery_Create_table_from_csv.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=BigQuery+-+Create+table+from+csv:+Error+short+description">🚨 Bug report</a>

**Tags:** #bigquery #database #snippet #operations #dataframe

**Author:** [Minura Punchihewa](https://www.linkedin.com/in/minurapunchihewa/)

This notebook loads data from a CSV file into a BigQuery table (the table will be created if it is not available already)

## Input

### Import libraries


```python
from naas_drivers import bigquery
```

### Variable


```python
service_account_file = 'big-query-777777-ffffff777777.json'
project_id = 'big-query-777777'
dataset = 'example_ds'
table_name = 'iris'
csv_file = 'iris.csv'
```

## Model

### Connect to BigQuery


```python
conn = bigquery.connect(service_account_file, project_id)
```

## Output

### Load data into the table


```python
conn.load_data_from_csv(dataset, table_name, csv_file)
```
