<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/AWS/AWS_Get_files_from_S3_bucket.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=AWS+-+Get+files+from+S3+bucket:+Error+short+description">🚨 Bug report</a>

**Tags:** #aws #cloud #storage #S3bucket #operations #snippet #url

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou/)

## Input

### Install packages


```python
!pip install boto3 getpass4
```

### Import library


```python
import boto3
```

### Variables


```python
ACCESS_KEY_ID = "**********"
SECRET_ACCESS_KEY = "**********"

BUCKET_NAME = "naas-example"
BUCKET_OBJECT_KEY = 'naas_happy_hour.mp3'
```

## Model

### Get file


```python
s3 = boto3.client('s3', aws_access_key_id=ACCESS_KEY_ID, aws_secret_access_key=SECRET_ACCESS_KEY)
fileObj = s3.get_object(Bucket=bucketname, Key=filename)
```

### Generate pre-signed URL


```python
file_url = s3.generate_presigned_url("get_object", Params={"Bucket": BUCKET_NAME, "Key": BUCKET_OBJECT_KEY}, ExpiresIn=604800)
```

## Output

### Display file


```python
fileOBJ
```

### Display pre-signed URL


```python
file_url
```
