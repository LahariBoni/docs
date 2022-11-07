<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Hugging%20Face/Hugging_Face_Question_Answering_from_PDF.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Hugging+Face+-+Question+Answering+from+PDF:+Error+short+description">Bug report</a>

**Tags:** #huggingface #ml #question_answer #ai #text

**Author:** [Muhammad Talha Khan](https://www.linkedin.com/in/muhtalhakhan/)

**Description**: This Transformers QA Pipeline shows a Question Answering models that can retrieve the answer to a question from a given text, which is useful for searching for an answer in a document. This specific example is set with the Bitcoin white paper but you can put any PDF.

The PDF will server as context which will used for questions answering.

## Input

### Install Packages


```python
!pip install tensorflow
```


```python
!pip install -q transformers
```

Use "--user" if it asks for permission prompt.


```python
!pip install PyPDF2
```


```python
!pip install urllib3
```

### Import Libraries



```python
from transformers import pipeline
import urllib.request
import PyPDF2
import io
```

### Add the Document Path


```python
URL = 'https://bitcoin.org/bitcoin.pdf'
req = urllib.request.Request(URL, headers={'User-Agent' : "Chrome"})
remote_file = urllib.request.urlopen(req).read()
remote_file_bytes = io.BytesIO(remote_file)
pdfdoc_remote = PyPDF2.PdfFileReader(remote_file_bytes)
```

You can change the URL path to the desired one relating to any of the PDF.

## Model

### Read Text from File


```python
pdf_text = ""    
    
for i in range(pdfdoc_remote.getNumPages()):
    print(i)
    page = pdfdoc_remote.getPage(i)
    page_content = page.extractText()
    pdf_text += page_content    
```

### Generate the text data from the pdf file 


```python
print(pdf_text)
```

### Loading the pipeline
Import Pipeline from Transformer after installing the transformers and tensorflow.


```python
nlp = pipeline('question-answering', model='deepset/roberta-base-squad2', tokenizer='deepset/roberta-base-squad2')
```

## Output

### Ask question


```python
context = pdf_text 
question = input('Enter your question:\n')

question_set = {
        'context': context,
        'question': question
    }

results = nlp(question_set)
```



### Get answer

This will print the answer to the question you have asked before.


```python
print("\nAnswer: " + results['answer'])
```