<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Snowflake/Snowflake_Basics_and_data_querying.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Snowflake+-+Basics+and+data+querying:+Error+short+description">🚨 Bug report</a>

**Tags:** #snowflake #data #warehouse #naas_drivers #snippet

**Author:** [Mateusz Polakowski](https://www.linkedin.com/in/polakowski/)

This notebook shows basic usage of Snowflake driver.

Below you can find essential operations for setting up the environment and querying data that already exists in your data warehouse.

## Input

### Import library


```python
import os
from naas_drivers import snowflake
from snowflake.connector.errors import ProgrammingError
```

### Setup Snowflake account

If you don't have your SF account, you can easily set up a [30-day trial account with $400 budget here](https://signup.snowflake.com/).

To get your Snowflake account ID essential for connecting, please refer to [Account Identifiers in Snowflake documentation](https://docs.snowflake.com/en/user-guide/admin-account-identifier.html). There are several methods to get your account ID, but the overall rule can be found below:

```<account_identifier>.snowflakecomputing.com```

If you're proceeding with the trial account, it's highly probable that your ID will resemble something like: `xy1234.eu-central-1`.

### Credentials


```python
# Here environment variables are used to pass Snowflake credentials, 
# but it's okay to do it in a different manner

sf_username=os.environ['SNOWFLAKE_USER']
sf_password=os.environ['SNOWFLAKE_PASSWORD']
sf_account=os.environ['SNOWFLAKE_ACCOUNT']
```

## Model

### Connecting to your Snowflake account


```python
snowflake.connect(
    username=sf_username,
    password=sf_password,
    account=sf_account
)
```

### Environment setup


```python
snowflake.database.get_current() is None
```


```python
snowflake.set_environment(
    warehouse='COMPUTE_WH',
    database='SNOWFLAKE_SAMPLE_DATA',
    schema='TPCH_SF100',
    role='ACCOUNTADMIN'
)
```


```python
snowflake.get_environment()
```

## Output

### Creating new database and schema


```python
snowflake.database.create('NAAS', or_replace=True)
```


```python
snowflake.database.use('NAAS', silent=True)
```


```python
snowflake.schema.create('INGESTION_SCHEMA', silent=True)
```


```python
snowflake.schema.use('INGESTION_SCHEMA', True)
```

### Executing custom query with a cursor


```python
snowflake.cursor.execute('SHOW SCHEMAS;').fetchall()
```

### Data querying - wrong schema


```python
snowflake.get_environment()
```


```python
query = 'SELECT * FROM CUSTOMER;'
```


```python
# Querying table that doesn't exist in NAAS/INGESTION_SCHEMA
try:
    results_1_not_working = snowflake.execute(query)
except ProgrammingError as pe:
    print('Something went wrong!')
    print(pe)
```

### Data querying - valid command run with session environment properly set up


```python
snowflake.set_environment(
    database = 'SNOWFLAKE_SAMPLE_DATA',
    schema = 'TPCH_SF100'
)
```


```python
results_1_working = snowflake.execute(query)
results_1_working
```

### Data querying - another valid command run


```python
results_2 = snowflake.execute(query, n=100)

print(f"Rows returned: {len(results_2['results'])}")
results_2['results'][:2]
```

### Data querying - mapping results to Pandas DataFrame


```python
results_pandas = snowflake.query_pd(query, n=100)

print(f'Rows returned: {len(results_pandas)}')
results_pandas.head(2)
```

## Extra

### Objects: `cursor` and `connection`

Both provided by Snowflake connector, that allow to execute any functionality possible with the original connector.


```python
snowflake.cursor.execute('SELECT CURRENT_WAREHOUSE()').fetchall()
```


```python
snowflake.connection.database
```
