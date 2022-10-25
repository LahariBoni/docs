<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Download_PDF_from_URL.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Download+PDF+from+URL:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #pdf #snippet #url #naas #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
import urllib
```

### Setup Variables


```python
# Input
pdf_path = "https://s22.q4cdn.com/959853165/files/doc_financials/2021/q4/da27d24b-9358-4b5c-a424-6da061d91836.pdf"

# Output
pdf_output = "2021 Annual Report.pdf"
```

## Model

### Download PDF


```python
def download_pdf(url, filepath):
    response = urllib.request.urlopen(url)    
    file = open(filepath, 'wb')
    file.write(response.read())
    file.close()
    print("File saved:", filepath) 
```

## Output

### Display result


```python
download_pdf(pdf_path, pdf_output)
```
