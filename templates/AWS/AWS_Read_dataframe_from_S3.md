<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/AWS/AWS_Read_dataframe_from_S3.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=AWS+-+Read+dataframe+from+S3:+Error+short+description">🚨 Bug report</a>

**Tags:** #aws #cloud #storage #S3bucket #operations #snippet #dataframe

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou/)

Reference : [AWS Data Wrangler](https://github.com/awslabs/aws-data-wrangler)

## Input

### Import libraries


```python
try:
    import awswrangler as wr
except:
    !pip install awswrangler --user
    import awswrangler as wr
```

### Setup AWS


```python
# Credentials
AWS_ACCESS_KEY_ID = "YOUR_AWS_ACCESS_KEY_ID"
AWS_SECRET_ACCESS_KEY = "YOUR_AWS_SECRET_ACCESS_KEY"
AWS_DEFAULT_REGION = "YOUR_AWS_DEFAULT_REGION"

# Bucket
BUCKET_PATH = f"s3://naas-data-lake/dataset/"
```

### Setup Env


```python
%env AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
%env AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
%env AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION
```

## Model

### Get dataframe


```python
df = wr.s3.read_parquet(BUCKET_PATH, dataset=True)
```

## Output

### Display result


```python
df
```
