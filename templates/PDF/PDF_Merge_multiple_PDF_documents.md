<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/PDF/PDF_Merge_multiple_PDF_documents.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=PDF+-+Merge+multiple++documents:+Error+short+description">🚨 Bug report</a>

**Tags:** #pdf #extract #snippet #operations #text

**Author:** [Minura Punchihewa](https://www.linkedin.com/in/minurapunchihewa/)

This notebook merges together multiple PDFs

## Input

### Import libraries


```python
import io
import requests
from urllib.parse import urlparse
try:
    from PyPDF2 import PdfFileReader, PdfFileWriter
except:
    !pip install PyPDF2 --user
    from PyPDF2 import PdfFileReader, PdfFileWriter
```

### Variables


```python
input_pdf_files = [
    "https://bitcoin.org/bitcoin.pdf",
    "https://ethereum.org/669c9e2e2027310b6b3cdce6e1c52962/Ethereum_Whitepaper_-_Buterin_2014.pdf"
]
ouput_pdf_file = "merge.pdf"
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

### Instantiate PDF writer


```python
pdf_writer = PdfFileWriter()
```

### Read PDF files and combine them


```python
for pdf_file in input_pdf_files:
    pdf_reader = get_pdf(pdf_file)
    
    for page_num in range(pdf_reader.numPages):
        page_obj = pdf_reader.getPage(page_num)
        pdf_writer.addPage(page_obj)
```

### Write the merged content into an output file


```python
pdf_output_file = open(ouput_pdf_file, 'wb')
pdf_writer.write(pdf_output_file)
pdf_output_file.close()
```
