<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_NLP_Examples.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+NLP+Examples:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #nlp #snippet #operations

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## How it works?
Naas NLP formulas follow this format.
```
nlp.get(task, model, tokenizer)(inputs)
```
The supported tasks are the following:

- text-generation (model: GPT2)
- summarization (model: t5-small)
- fill-mask (model: distilroberta-base)
- text-classification (model: distilbert-base-uncased-finetuned-sst-2-english)
- feature-extraction (model: distilbert-base-cased)
- token-classification (model: dslim/bert-base-NER)
- question-answering
- translation

We use [Hugging Face API](https://huggingface.co/models) under the hood to access the models.

## Input

### Import library


```python
from naas_drivers import nlp
```

## Model

### Text Generation


```python
nlp.get("text-generation", model="gpt2", tokenizer="gpt2")("What is the most important thing in your life right now?")
```

### Text Summarization
Summarize the text given, maximum lenght (number of tokens/words) is set to 200.


```python
nlp.get("summarization", model="t5-small", tokenizer="t5-small")('''

There will be fewer and fewer jobs that a robot cannot do better. 
What to do about mass unemployment this is gonna be a massive social challenge and 
I think ultimately we will have to have some kind of universal basic income.

I think some kind of a universal basic income is going to be necessary 
now the output of goods and services will be extremely high 
so with automation they will they will come abundance there will be or almost everything will get very cheap.

The harder challenge much harder challenge is how do people then have meaning like a lot of people 
they find meaning from their employment so if you don't have if you're not needed if 
there's not a need for your labor how do you what's the meaning if you have meaning 
if you feel useless these are much that's a much harder problem to deal with. 

''')
```

### Text Classification
Basic sentiment analysis on a text.<br>
Returns a "label" (negative/neutral/positive), and score between -1 and 1.


```python
nlp.get("text-classification", 
        model="distilbert-base-uncased-finetuned-sst-2-english",
        tokenizer="distilbert-base-uncased-finetuned-sst-2-english")('''

It was a weird concept. Why would I really need to generate a random paragraph? 
Could I actually learn something from doing so? 
All these questions were running through her head as she pressed the generate button. 
To her surprise, she found what she least expected to see.

''')
```

### Fill Mask

Fill the blanks ('< mask >') in a sentence given with multiple proposals. <br>
Each proposal has a score (confidence of accuracy), token value (proposed word in number), token_str (proposed word)


```python
nlp.get("fill-mask",
        model="distilroberta-base",
        tokenizer="distilroberta-base")('''

It was a beautiful <mask>.

''')
```

### Feature extraction
This generate a words embedding (extract numbers out of the text data).<br>
Output is a list of numerical values.


```python
nlp.get("feature-extraction", model="distilbert-base-cased", tokenizer="distilbert-base-cased")("Life is a super cool thing")
```

### Token classification
Basically NER. If you give names, location, or any "entity" it can detect it.<br>

| Entity abreviation | Description                                                                  |
|--------------|------------------------------------------------------------------------------|
| O            | Outside of a named entity                                                    |
| B-MIS        | Beginning of a miscellaneous entity right after another miscellaneous entity |
| I-MIS        | Miscellaneous entity                                                         |
| B-PER        | Beginning of a person’s name right after another person’s name               |
| I-PER        | Person’s name                                                                |
| B-ORG        | Beginning of an organization right after another organization                |
| I-ORG        | organization                                                                 |
| B-LOC        | Beginning of a location right after another location                         |
| I-LOC        | Location                                                                     |


Full documentation : https://huggingface.co/dslim/bert-base-NER.<br>

## Output

### Display result


```python
nlp.get("token-classification", model="dslim/bert-base-NER", tokenizer="dslim/bert-base-NER")('''

My name is Wolfgang and I live in Berlin

''')
```
