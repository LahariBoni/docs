<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Snowflake/Snowflake_Ingest_json_data_from_local_stage.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Snowflake+-+Ingest+json+data+from+local+stage:+Error+short+description">🚨 Bug report</a>

**Tags:** #snowflake #data #warehouse #naas_drivers #snippet

**Author:** [Mateusz Polakowski](https://www.linkedin.com/in/polakowski/)

To use your own data inside Snowflake DWH, first thing first, you need to ingest it.

This notebook shows how to put <u>JSON</u> data from your local machine into Snowflake. Several objects and actions will be necessary to do so:

- creating file format,
- creating internal named stage (for other stages, please check [Snowflake documentation](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-create-stage.html)),
- putting file into stage,
- copying data from stage to the table.

All above steps are essential to make use of the data from your local machine inside Snowflake.

You can try with your own JSON data, but to follow the notebook cells 1:1 please download `reviews_dev.json` file from `awesome-notebooks/Snowflake` directory. Eventually, you may need to change your file location below in the ensuing cells.

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
    role='ACCOUNTADMIN'
)
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
# It's possible to add extra parameters to the function
# They will be passed as additional variables at the end of the query statement
results_ff = snowflake.file_format.create(
    'my_json_format', 
    'JSON', 
    or_replace=True,
    TRIM_SPACE = 'TRUE'
)

results_ff
```


```python
# Let's not forget about `snowflake.execute` functionality
snowflake.execute('SHOW FILE FORMATS;')['results']
```

### Creating file format with wrong parameter (not applicable for JSON file format, see [docs](https://docs.snowflake.com/en/sql-reference/sql/create-file-format.html))


```python
try:
    snowflake.file_format.create('my_invalid_son_format', 'JSON', or_replace=True, ESCAPE = '"\\"')
except ProgrammingError as pe:
    print('Something went wrong!')
    print(pe)
```

### Dropping file format


```python
# snowflake.file_format.drop('my_json_format', if_exists=True)
```

### Creating internal stage


```python
snowflake.stage.create(
    stage_name='my_internal_stage', 
    or_replace=True,
    file_format_name='my_json_format'
)
```

### Putting data on the stage


```python
snowflake.stage.put(
    filepath='file://~/Downloads/reviews_data.json',
    internal_stage_name='@my_internal_stage',
    silent=True
)
```

### Listing files inside stage


```python
result_list_stage = snowflake.stage.list('@my_internal_stage')

print(result_list_stage['results'])
print(result_list_stage['statement'])
```

### Creating table with query execution


```python
query_create_table = """
    CREATE OR REPLACE TABLE reviews_dev (
      language VARCHAR
      , product_category VARCHAR
      , product_id VARCHAR
      , review_body VARCHAR
      , review_id VARCHAR
      , review_title VARCHAR
      , reviewer_id VARCHAR
      , stars INT
    );
"""

# No worries, Table API will be available soon too!
snowflake.execute(query_create_table)['results']
```

### Loading data from internal stage to a table

As we're loading data in JSON format, transformations are required to not put everything into a single VARIANT type column (for more, [see the documentation](https://docs.snowflake.com/en/sql-reference/data-types-semistructured.html)).


```python
transformation_statement = """
    SELECT
        $1:language::varchar AS language
        , $1:product_category::varchar AS product_category
        , $1:product_id::varchar AS product_id
        , $1:review_body::varchar AS review_body
        , $1:review_id::varchar AS review_id
        , $1:review_title::varchar AS review_title
        , $1:reviewer_id::varchar AS reviewer_id
        , $1:stars::int AS stars
    FROM @my_internal_stage
"""

snowflake.copy_into(
    table_name='reviews_dev',
    source_stage=transformation_statement,
    silent=True
)
```

### Data querying straight to Pandas DataFrame


```python
reviews_dev = snowflake.query_pd('SELECT * FROM reviews_dev')
reviews_dev.head()
```
