<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Notion/Notion_Automate_transcript_generation_from_recording_link_in_page_property.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Notion+-+Automate+transcript+generation+from+recording+link+in+page+property:+Error+short+description">🚨 Bug report</a>

**Tags:** #notion #aws #transcribe #S3 #automation

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

This notebook is used to automatically add to a notion page, the transcript of a recording that would be found in the `Recording` property of that same page.

Recordings can be in any language compatible with AWS transcribe ([Here is the list](https://docs.aws.amazon.com/transcribe/latest/dg/supported-languages.html))

As of today, you also need to upload the recordings on Google Drive. (Feel free to open an issue [here](https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Notion%20-%20Automatic%20transcribe%20recording%20-%20Add%20Zoom%20recording%20sources) if you want more recording sources like Zoom, etc to be developed.)

To setup this Notebook you need to have:

1. An AWS Account + an IAM user having the following permissions:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "s3-object-lambda:*"
            ],
            "Resource": "*"
        }
    ]
}
```
_⚠️ This IAM permission is too broad, you should limit it to a single S3 bucket._

and

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "transcribe:*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::*transcribe*"
            ]
        }
    ]
}
```
2. An AWS S3 bucket where we will be able to push video files and get JSON transcripts.
3. A Notion integration that will have read/write access to your Notion database.
    1. [Get your Notion integration token](https://docs.naas.ai/drivers/notion)

## Input

### Imports


```python
# Used to access nested properties in dictionnaries.
import pydash as _

# Used to run multiple jobs at once.
import threading, queue
import uuid

# Used for nicer cell outputs.
try:
    from rich import print
except:
    ! pip install --user rich
    from rich import print

# Used for transcript generation.
import sys
import json
import datetime
import codecs
import time
import os

# Used for email rendering.
import markdown

# Used to download files from google drive.
try:
    import gdown
except:
    !pip install gdown
    import gdown

# Used to interact with AWS, here S3 and Transcribe.
try:
    import boto3
except:
    ! pip install --user boto3
    
# Used to interact with Notion.
from naas_drivers import notion
from naas_drivers.tools.notion import BlockTypeFactory

# Used to schedule, send notification, get/store secrets.
import naas
```

### Input parameters

- [Get your Notion integration token](https://docs.naas.ai/drivers/notion) (Once you have it you also need to share your database with this integration.)


```python
# Notion secret
NOTION_TOKEN = naas.secret.get('tf_notion_token')
NOTION_DATABASE_ID = "your notion database id"

# AWS Secret.
# IAM permissions to S3 and Transcribe are needed to have this template working.
AWS_ACCESS_KEY_ID = naas.secret.get('aws_transcribe_s3_key')
AWS_SECRET_ACCESS_KEY = naas.secret.get('aws_transcribe_s3_secret')
AWS_REGION = 'eu-west-3'

# AWS S3 Bucket name that will be used to send recording and fetch transcripts.
BUCKET_NAME = "your-existing-s3-bucket"

# To whom we need to send notification errors.
EMAIL_ERROR_TO = "maxime@naas.ai"
```

### Setup Notion


```python
notion.connect(NOTION_TOKEN)
```

## Model

### Load database


```python
database = notion.connect(NOTION_TOKEN).databases.get(NOTION_DATABASE_ID)
```

### Error notifier


```python
def send_error_notification(notion_page, error):
    email_to = EMAIL_ERROR_TO
    subject = "🛎️ Transcript error 🚨"
    content = markdown.markdown(
f'''
# 🚨 An error occured while generating transcript.

Notion page: {notion_page}

## Error:

> {error}

''')
    
    naas.notifications.send(email_to, subject, content)
```

### Load notion pages containing Recordings


```python
pages = database.query({
    "filter": {
        "and": [
        {
            "property": "Transcript computed",
            "select": {
                "is_empty": True
            }
        },
        {
            "property": "Recording",
            "url": {
                "is_not_empty": True
            }
        }
    ]
    }
})
len(pages)
```

### Download recording


```python
class UnknownSource(Exception):
    """This class is an exception used to identify the fact that we do not know how
    to download a recording.
    """
    pass

def download_recording(url, out):
    
    if 'drive.google.com' in url:
        gdown.download(url, out, quiet=False, fuzzy=True)
    else:
        raise UnknownSource('Do not know how to download recording.')
```

### Upload to S3


```python
def upload_to_s3(filename):
    s3 = boto3.client('s3', aws_access_key_id=AWS_ACCESS_KEY_ID, aws_secret_access_key=AWS_SECRET_ACCESS_KEY)

    with open(filename, "rb") as f:
        s3.upload_fileobj(f, BUCKET_NAME, filename)
```

### Create transcribe job


```python
def create_transcribe_job(job_name, filename, media_format='mp4'):
    
    transcribe = boto3.client('transcribe', aws_access_key_id=AWS_ACCESS_KEY_ID, aws_secret_access_key=AWS_SECRET_ACCESS_KEY, region_name=AWS_REGION)
    result = transcribe.start_transcription_job(TranscriptionJobName=job_name,
        MediaFormat=media_format,
        Media={
            'MediaFileUri': 's3://{0}/{1}'.format(BUCKET_NAME, filename)
        },
        Settings={
            'ShowSpeakerLabels': True,
            'MaxSpeakerLabels': 5
        },
        OutputBucketName=BUCKET_NAME,
        OutputKey=f'{filename}.json',
        IdentifyLanguage=True)
    return result
```

### Get transcription job status


```python
def get_transcribe_job_status(job_name):    
    transcribe = boto3.client('transcribe', aws_access_key_id=AWS_ACCESS_KEY_ID, aws_secret_access_key=AWS_SECRET_ACCESS_KEY, region_name='eu-west-3')
    result = transcribe.get_transcription_job(TranscriptionJobName=f'{job_name}')
    return result
```

### Download from s3


```python
def download_from_s3(object_name):
    s3 = boto3.client('s3', aws_access_key_id=AWS_ACCESS_KEY_ID, aws_secret_access_key=AWS_SECRET_ACCESS_KEY)
    local_store_path = os.path.join('/', 'tmp', object_name)
    with open(local_store_path, 'wb') as f:
        s3.download_fileobj(BUCKET_NAME, object_name, f)
        return local_store_path
    return ''
```

### Convert transcript


```python
def convert_transcript(filename):
    with codecs.open(filename+'.txt', 'w', 'utf-8') as w:
        with codecs.open(filename, 'r', 'utf-8') as f:
            data=json.loads(f.read())
            labels = data['results']['speaker_labels']['segments']
            speaker_start_times={}
            for label in labels:
                for item in label['items']:
                    speaker_start_times[item['start_time']] =item['speaker_label']
            items = data['results']['items']
            lines=[]
            line=''
            time=0
            speaker='null'
            i=0
            for item in items:
                i=i+1
                content = item['alternatives'][0]['content']
                if item.get('start_time'):
                    current_speaker=speaker_start_times[item['start_time']]
                elif item['type'] == 'punctuation':
                    line = line+content
                if current_speaker != speaker:
                    if speaker:
                        lines.append({'speaker':speaker, 'line':line, 'time':time})
                    line=content
                    speaker=current_speaker
                    time=item['start_time']
                elif item['type'] != 'punctuation':
                    line = line + ' ' + content
            lines.append({'speaker':speaker, 'line':line,'time':time})
            sorted_lines = sorted(lines,key=lambda k: float(k['time']))
            for line_data in sorted_lines:
                line='[' + str(datetime.timedelta(seconds=int(round(float(line_data['time']))))) + '] ' + line_data.get('speaker') + ': ' + line_data.get('line')
                w.write(line + '\n\n')
            return filename+'.txt'
```

### Text to chunks


```python
def text_to_chunks(text, chunk_size):
    chunks = []
    count = 0
    text_len = len(text)
    
    while count * chunk_size < text_len:
        chunks.append(text[count * chunk_size:count * chunk_size + chunk_size])
        count += 1
    return chunks
```

### Handle Job


```python
def handle_job(job):
    print(f'Handling Job: {job}')
    recording_url = job.recording_url
    notion_page_id = job.notion_page_id

    # Download recording
    download_location = f'{notion_page_id}'
    download_recording(recording_url, download_location)
    print(f'Downloaded file to {download_location}')
    
    # Upload file to s3.
    upload_to_s3(download_location)
    print(f'File uploaded to s3.')

    # Create transcribe job.
    result = create_transcribe_job(job.id, download_location)
    print(f'Transcription job created.')

    # Wait for transcription job to complete.
    waiting_for_job = True
    job_status = None
    transcript = ""
    while waiting_for_job:
        status = get_transcribe_job_status(job.id)
        job_status = _.get(status, 'TranscriptionJob.TranscriptionJobStatus')
        
        
        print(f'Waiting for job. Actual status "{job_status}"')
        if job_status == "COMPLETED":
            print(f'{job}: AWS Transcribe job completed')
            waiting_for_job = False

            media_file_uri = _.get(status, 'TranscriptionJob.Media.MediaFileUri')
            transcript_file_uri = _.get(status, 'TranscriptionJob.Transcript.TranscriptFileUri')
            
            
            local_store_path = download_from_s3(transcript_file_uri.split('/')[-1])
            print(f'Transcript downloaded {local_store_path}')
            converted_transcript_path = convert_transcript(local_store_path)
            
            
            with open(converted_transcript_path, 'r') as tr:
                for line in tr.readlines():
                    transcript = transcript + line
            
            os.remove(local_store_path)
            os.remove(converted_transcript_path)
            
        elif job_status == "IN_PROGRESS":
            print(f'Job in progress')
            time.sleep(5)

        elif job_status not in ['COMPLETED', 'IN_PROGRESS', 'QUEUED']:
            waiting_for_job = False
            print(f'This job might be in error state.')
            print(_.get(status, 'TranscriptionJob.FailureReason'))
            raise Exception(f'Error while running aws transcribe. {_.get(status, "TranscriptionJob.FailureReason")}')
    
    if transcript != '' and job_status == "COMPLETED":
        notion_page_id = notion_page_id
        page = notion.page.get(notion_page_id)
        print(f'Adding toggle.')
        page.toggle('Transcript')
        page.update()
        for block in page.get_blocks():
            if block.type == 'toggle' and _.get(block, 'toggle.text[0].plain_text') == 'Transcript':
                block_id = block.id
                children_blocks = []
                for chunk in text_to_chunks(transcript, 2000):
                    child = BlockTypeFactory.new_default(type='paragraph', payload=chunk)
                    children_blocks.append(child)
                    
                notion.block.append(block_id, children_blocks)
                page.select("Transcript computed", "True")
                page.update()
                print(f'Transcript added to notion page {notion_page_id}.')
```

### Create Job class


```python
class Job:
    def __init__(self, notion_page_id, recording_url, job_id=None):
        self.notion_page_id = notion_page_id
        self.recording_url = recording_url
        self.id = job_id if job_id else str(uuid.uuid4())
    
    def __str__(self):
        return f'Job ID = {self.id} Notion Page Id = {self.notion_page_id} Recording URL = {self.recording_url}'
```

## Output

### Handle all recordings


```python
for page in pages:
    recording_url = _.get(page.properties, 'Recording.url')
    
    if recording_url != None:
        job = Job(page.id, recording_url)
        try:
            handle_job(job)
        except Exception as e:
            page = notion.page.get(job.notion_page_id)
            
            send_error_notification(page.url, str(e))   
            page.select("Transcript computed", "ERROR")
            page.update()

```

# Schedule this job


```python
naas.scheduler.add(cron="0 * * * *")
```
