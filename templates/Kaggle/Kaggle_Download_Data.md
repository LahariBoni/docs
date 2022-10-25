<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Kaggle/Kaggle_Download_Data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Kaggle+-+Download+Data:+Error+short+description">🚨 Bug report</a>

**Tags:** #kaggle #dataset #download #data #datascience

**Author:** [Muhammad Waqar Gul](https://www.linkedin.com/in/waqar-gul)

This notebook allows you to directly download the datasets from Kaggle inside your notebook directory, without the need of downloading them manually.

**Tutorial Link**: https://www.loom.com/share/440e07838b534fe28f95fc57ae6cab89

## Input

### Import libraries


```python
try:
    import opendatasets as od
    import pandas as pd
except:
    !pip install opendatasets
    import opendatasets as od
from os import path
```

### Variables


```python
url = ''                    ### kaggle dataset url here
data_dir = ''               ### directory where you want to save data
```

## Model


    

When the function is run, it is going to ask you for your Kaggle username and your Kaggle key. 

For getting access to the Kaggle username and key, inside your Kaggle profile:

- Go to the account tab and under API section, click Create New API Token. 

- A JSON file will be downloaded, open it locally or you can also use any online JSON viewer and upload it there. 

- On opening this file, you will find the username and key in it. Copy the username and  password and paste it into the prompted Notebook cell. The content of the downloaded file would look like this.

                    {"username":<KAGGLE USERNAME>,"key":<KAGGLE KEY>}
    
    
<b>*Note: Your Kaggle key is private, make sure that you dont share it with anyone else.    </b>

### Function


```python
"""
    function for downloading data, pass the URL of the dataset and directory where to save
    as an argument to the function.
"""

def download_data(url, data_dir): 
    od.download(url, data_dir)
    
download_data(url, data_dir) 
```

## Output

### Display CSV result
Downloaded file path csv needs to be set manually.


```python
downloaded_file_path_csv = ""
if path.exists(downloaded_file_path_csv):
    df = pd.read_csv(downloaded_file_path_csv) # pass downloaded filepath as an argument here
    df.head()
```
