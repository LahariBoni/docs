<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/PDF/PDF_Extract_Text_from_PDF.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=PDF+-+Extract+Text+from:+Error+short+description">🚨 Bug report</a>

**Tags:** #pdf #extract #snippet #operations #text

**Author:** [Minura Punchihewa](https://www.linkedin.com/in/minurapunchihewa/)

This notebook extract the text from a PDF.

## Input

### Import libraries


```python
import io
import requests
from urllib.parse import urlparse
try:
    from PyPDF2 import PdfFileReader, PdfFileWriter
except:
    !pip install PyPDF2
    from PyPDF2 import PdfFileReader, PdfFileWriter
```

### Variables


```python
pdf_file = "https://ethereum.org/669c9e2e2027310b6b3cdce6e1c52962/Ethereum_Whitepaper_-_Buterin_2014.pdf"
```

## Model

### Define function to validate if string is a valid URL


```python
def is_url(url):
    try:
        result = urlparse(url)
        return all([result.scheme, result.netloc])
    except ValueError:
        return False
```

### Define function to read PDF file


```python
def get_pdf(path):
    if is_url(path):
        remote_file = requests.get(path).content
        memory_file = io.BytesIO(remote_file)
        pdf_file = PdfFileReader(memory_file)
    else:
        pdf_file_obj = open(path, 'rb')
        pdf_file = PdfFileReader(pdf_file_obj)
        
    return pdf_file
```

## Output

### Read PDF file and extract text


```python
texts = []
pdf_reader = get_pdf(pdf_file)    
    
for page_num in range(pdf_reader.numPages):
    page_obj = pdf_reader.getPage(page_num)
    texts.append(page_obj.extractText())
```

### Display text


```python
extract_texts = ''.join(texts)
print(extract_texts)
```
