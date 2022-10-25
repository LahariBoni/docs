<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Snowflake/Snowflake_Ingest_data_from_AWS_external_stages.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Snowflake+-+Ingest+data+from+AWS+external+stages:+Error+short+description">🚨 Bug report</a>

**Tags:** #snowflake #data #warehouse #naas_drivers #snippet

**Author:** [Mateusz Polakowski](https://www.linkedin.com/in/polakowski/)

To use your own data inside Snowflake DWH, first thing first, you need to ingest it.

This notebook shows how to ingest data from AWS S3 to Snowflake. Several objects and actions will be necessary to do so:

- creating file format,
- creating external stage for public AWS S3 bucket,
- creating storage integration,
- creating external stage for private AWS S3 bucket using storage integration,
- copying data from stages to the tables.

You can try with your own data (as well as other file formats), but to follow the notebook cells 1:1 please download `reviews_data.json` file from `awesome-notebooks/Snowflake` directory and put it somewhere inside your private AWS account. 

For fetching data from public AWS S3 bucket (`s3://amazon-reviews-ml/json/dev/dataset_en_dev.json`), you don't need any permission, mentioned bucket's URI is sufficient.

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
snowflake.role.use('ACCOUNTADMIN', silent=True)
snowflake.warehouse.use('COMPUTE_WH', silent=True)
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
    'my_json_format', 
    'JSON',
    or_replace=True
)

results_ff
```

### Creating external stage for public data


```python
snowflake.stage.create(
    stage_name='external_aws_stage_public',
    or_replace=True,
    file_format_name='my_json_format',
    url="'s3://amazon-reviews-ml/json/dev/dataset_en_dev.json'"
)
```

### Creating a table


```python
query_create_table = """
    CREATE OR REPLACE TABLE reviews_dev_public (
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

### Loading data from public external stage to a table

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
    FROM @external_aws_stage_public
"""

snowflake.copy_into(
    table_name='reviews_dev_public',
    source_stage=transformation_statement,
    silent=True
)
```

### Data querying straight to Pandas DataFrame


```python
reviews_dev_public = snowflake.query_pd('SELECT * FROM reviews_dev_public')
reviews_dev_public.head()
```

---

### Creating storage integration for AWS S3 bucket connection with Snowflake

For more about connecting your private cloud storage with Snowflake see below documentation pages:

- [Loading data to Snowflake from AWS S3](https://docs.snowflake.com/en/user-guide/data-load-s3.html)
- [Loading data to Snowflake from Google Cloud Storage](https://docs.snowflake.com/en/user-guide/data-load-gcs.html)
- [Loading data to Snowflake from Azure](https://docs.snowflake.com/en/user-guide/data-load-azure.html)
- [External stage create command - necessary parameters](https://docs.snowflake.com/en/sql-reference/sql/create-stage.html#external-stage-parameters-externalstageparams)

Bear in mind that **only users with ACCOUNTADMIN role** selected can create storage integration objects.

Below cells follow `Option 1` from the [list of available approaches](https://docs.snowflake.com/en/user-guide/data-load-s3-config.html). It's highly recommended to do it this way, thus let's follow the advice.

Some of the parameters are dummy, for security reasons, although cells were executed with proper values. If you follow the instructions in Snowflake documentation, you shouldn't have any issues modifying them.


```python
snowflake.storage_integration.create(
    storage_integration_name='storage_integration_aws_naas_sf_data',
    storage_provider='S3',
    storage_allowed_locations=['s3://naas-snowflake-data/'],
    STORAGE_AWS_ROLE_ARN="'arn:aws:iam::112233445566:role/NaasDummyRole'",
    or_replace=True,
    silent=True
)
```

### Setting up proper IAM access

Policy code is taken directly from [Snowflake documentation](https://docs.snowflake.com/en/user-guide/data-load-s3-config-storage-integration.html).


```python
import pandas as pd
pd.set_option('display.max_colwidth', None) # useful to copy-paste parameters

# Remove `head()` to see all necessary values
snowflake.query_pd('DESC INTEGRATION storage_integration_aws_naas_sf_data;').iloc[:, [0, 2]].head(2)
```

According to values returned by storage integration description, please go to AWS console and alter your IAM Role's Trust Policy.

When this is done, you can proceed forward.

### Creating external stage for private data


```python
snowflake.stage.create(
    stage_name='external_aws_stage_private',
    STORAGE_INTEGRATION='storage_integration_aws_naas_sf_data',
    url="'s3://naas-snowflake-data/'",
    file_format_name='my_json_format',
    or_replace=True
)
```

### Listing external stage objects


```python
snowflake.query_pd('LIST @external_aws_stage_private')
```

### Creating a table


```python
query_create_table = """
    CREATE OR REPLACE TABLE reviews_dev_private (
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

snowflake.execute(query_create_table)['results']
```

### Loading data from private external stage to a table


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
    FROM @external_aws_stage_private
"""

snowflake.copy_into(
    table_name='reviews_dev_private',
    source_stage=transformation_statement,
    silent=True
)
```

### Querying data straight to Pandas DataFrame


```python
pd.set_option('display.max_colwidth', 100)
reviews_dev_public = snowflake.query_pd('SELECT * FROM reviews_dev_private')
reviews_dev_public.head()
```

### Dropping storage integration


```python
snowflake.storage_integration.drop('storage_integration_aws_naas_sf_data')
```
