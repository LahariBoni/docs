<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Redshift/Redshift_Connect_with_SQL_Magic_and_IAM_Credentials.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Redshift+-+Connect+with+SQL+Magic+and+IAM+Credentials:+Error+short+description">🚨 Bug report</a>

**Tags:** #redshift #database #snippet #operations #naas #jupyternotebooks

**Author:** [Caleb Keller](https://www.linkedin.com/in/calebmkeller/)

## Input

- ipython-sql
- boto3
- psycopg2
- sqlalchemy-redshift

If you're running in NaaS, you can execute the below to install the necessary libraries.


```python
!pip install -q ipython-sql boto3 psycopg2-binary sqlalchemy-redshift
```

When using the `ipython-sql` library of *SQL Magics* I like to put the `%reload_ext sql` at the top. This loads the extension if it isn't already loaded or reloads it. Reload instead of load helps it not error out if you so something like run all cells.


```python
%reload_ext sql

import boto3
import psycopg2
import getpass
import pandas as pd
from urllib import parse

```

## Model

The SQL Magic, powered by SQL ALchemy, needs the connection string in a specific format. The below function does several things:

1. It takes in your AWS IAM credentials.
2. It uses those credentials to get temporary database credentials.
3. It creates a SQL alchemy connection string from those credentials.



```python
def rs_connect(dbname, dbhost, clusterid, dbport, dbuser, region_name='us-east-1'):

    ''' Connect to redshift using AIM credentials'''
    aaki = getpass.getpass('aws_access_key_id')
    asak = getpass.getpass('aws_secret_access_key')
    aws_session = boto3.session.Session(aws_access_key_id=aaki, aws_secret_access_key=asak, region_name=region_name)
    aaki = ''; asak = ''
    
    aws_rs = aws_session.client('redshift')
    response = aws_rs.get_cluster_credentials(DbUser=dbuser, DbName=dbname, ClusterIdentifier=clusterid, AutoCreate=False)
    
    ''' Convert those credentials into Database user credentials '''
    dbuser = response['DbUser']
    dbpwd = response['DbPassword']
    
    ''' Generate the SQLAlchemy Connection string '''
    connectionString = 'redshift+psycopg2://{username}:{password}@{host}:{port}/{db}?sslmode=prefer'.format(username=parse.quote_plus(dbuser), password=parse.quote_plus(dbpwd), host=dbhost, port=dbport, db=dbname)

    dbuser = None; dbpwd = None; conn_str = None; response = None;
    
    return connectionString

```

## Output

Run the below and replace the parameters with your own server's information.


```python
connectionString = rs_connect('database_name', 'host', 'cluster_id', 5439, 'database_user')

```


```python
%sql $connectionString

```


```sql
%%sql

select
    your,
    sql,
    goes,
    here
from
  your.brain

```


Article from the author here: <a href="https://calebmkeller.medium.com/jupyter-sql-magic-connection-to-redshift-using-iam-credentials-8a9c53ce29db" _target="blank">here</a>.

For more on SQL magics read up on them with the below links:
 - https://towardsdatascience.com/jupyter-magics-with-sql-921370099589
 - https://blog.dominodatalab.com/lesser-known-ways-of-using-notebooks/
