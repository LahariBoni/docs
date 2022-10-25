<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/ZIP/ZIP_Extract_files.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=ZIP+-+Extract+files:+Error+short+description">🚨 Bug report</a>

**Tags:** #zip #extract #file #operations #snippet #naas

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

## Input

### Import libraries


```python
import zipfile
from io import BytesIO
import re
```

## Model

### Function


```python
def extract_zip(filepath):
    i = 0
    with zipfile.ZipFile(filepath, "r") as zfile:
        for name in zfile.namelist():
            if re.search(r'\.zip$', name) is not None:
                zfiledata = BytesIO(zfile.read(name))
                with zipfile.ZipFile(zfiledata) as zfile2:
                    for name2 in zfile2.namelist():
                        zfile2.extract(name2, path="../", pwd=None)
                        i=i+1
    zfile.close()
    print("Processing Completed. "+str(i)+" file(s) extracted")
```

## Output

### Extract zip


```python
extract_zip('bilans_saisis_20181231.zip')
```
