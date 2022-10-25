<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Snowflake/Snowflake_Ingest_csv_data_from_local_stage.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Snowflake+-+Ingest+csv+data+from+local+stage:+Error+short+description">🚨 Bug report</a>

**Tags:** #snowflake #data #warehouse #naas_drivers #snippet

**Author:** [Mateusz Polakowski](https://www.linkedin.com/in/polakowski/)

To use your own data inside Snowflake DWH, first thing first, you need to ingest it.

This notebook shows how to put <u>CSV</u> data from your local machine into Snowflake. Several objects and actions will be necessary to do so:

- creating file format,
- creating internal named stage (for other stages, please check [Snowflake documentation](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-create-stage.html)),
- putting file into stage,
- copying data from stage to the table.

All above steps are essential to make use of the data from your local machine inside Snowflake.

You can try with your own JSON data, but to follow the notebook cells 1:1 please download `sales_data.csv` file from `awesome-notebooks/Snowflake` directory. Eventually, you may need to change your file location below in the ensuing cells.

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
snowflake.warehouse.use('COMPUTE_WH')
```


```python
snowflake.role.use('ACCOUNTADMIN')
```


```python
snowflake.database.create('NAAS', or_replace=True, silent=True)
snowflake.database.use('NAAS', silent=True)
snowflake.schema.create('NAAS_SCHEMA', or_replace=True, silent=True)
snowflake.schema.use('NAAS_SCHEMA', silent=True)
```


```python
snowflake.get_environment()
```

## Output

### Creating file format


```python
results_ff = snowflake.file_format.create(
    'my_csv_format', 
    'CSV', 
    or_replace=True,
    FIELD_DELIMITER="','",
    SKIP_HEADER=1
)

results_ff
```

### Creating file format with wrong parameter (not applicable for JSON file format, see [docs](https://docs.snowflake.com/en/sql-reference/sql/create-file-format.html))


```python
try:
    snowflake.file_format.create('my_invalid_csv_format', 'CSV', or_replace=True, ENABLE_OCTAL = "TRUE")
except ProgrammingError as pe:
    print('Something went wrong!')
    print(pe)
```

### Dropping file format


```python
# snowflake.file_format.drop('my_csv_format', if_exists=True)
```

### Creating internal stage


```python
snowflake.stage.create(
    stage_name='my_internal_stage', 
    or_replace=True,
    file_format_name='my_csv_format'
)
```

### Putting data on the stage


```python
snowflake.stage.put(
    filepath='file://~/Downloads/sales_data.csv',
    internal_stage_name='@my_internal_stage',
    silent=True
)
```

### Showing files inside stage


```python
result_list_stage = snowflake.stage.list('@my_internal_stage')

print(result_list_stage['results'])
```

### Creating a table

It's possible to do something like `CREATE TABLE AS SELECT ... FROM @stage`, although it's highly not recommended! With that approach we don't keep file loading history (which Snowflake is capable to do and make a lot of use from).


```python
query_create_table = """
    CREATE OR REPLACE TABLE sales_data (
      month VARCHAR
      , year INT
      , item VARCHAR
      , amount BIGINT
      , currency VARCHAR
    );
"""

# No worries, Table API will be available soon too!
snowflake.execute(query_create_table)['results']
```

### Loading data from internal stage to a table


```python
snowflake.copy_into(
    table_name='sales_data',
    source_stage='@my_internal_stage',
    silent=True
)
```

### Running the same command again 

No harm as `COPY INTO` tracks which files have been loaded already


```python
snowflake.copy_into(
    table_name='sales_data',
    source_stage='@my_internal_stage',
    silent=True
)
```

### Data querying straight to Pandas DataFrame


```python
sales_data = snowflake.query_pd('SELECT * FROM sales_data')
sales_data.head()
```
